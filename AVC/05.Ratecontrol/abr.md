# Rate Control - ABR


## 总体流程

![overall](./assets/overall.png)

## rc初始化

在open encoder的时候会初始化，根据**h->param.rc**中的参数，初始化**h->rc**(**x264_ratecontrol_t**)

h->param.rc中几个重要的默认参数：

```c
h->param.rc.i_qp_constant = -1;
h->param.rc.i_qp_min = 0;
h->param.rc.i_qp_max = 69;
h->param.rc.i_qp_step = 4;              // 两帧间qp的最大变化幅度
h->param.rc.f_ip_factor = 1.4;
h->param.rc.f_pb_factor = 1.0;          // 开启mb_tree后pb_factor强制为1.0
h->param.rc.f_rf_constant = 23.0;
h->param.rc.f_rate_tolerance = 1.0;
h->param.rc.f_vbv_buffer_init: 0.9;
h->param.rc.f_qcompress = 0.6;
h->param.rc.f_qblur = 0.5;
h->param.rc.f_complexity_blur = 20.0;
```

h->rc中几个重要的默认参数：

```c
h->rc->qcompress = 1; // 开启mbtree后rc->qcompress强制为0
// 初始QP
#define ABR_INIT_QP 24 
rc->accum_p_norm = .01;
rc->accum_p_qp = ABR_INIT_QP * rc->accum_p_norm;

rc->buffer_fill_final =
rc->buffer_fill_final_min = rc->buffer_size * h->param.rc.f_vbv_buffer_init;

rc->qp_constant[SLICE_TYPE_P] = h->param.rc.i_qp_constant;
rc->qp_constant[SLICE_TYPE_I] = h->param.rc.i_qp_constant - rc->ip_offset;
rc->qp_constant[SLICE_TYPE_B] = h->param.rc.i_qp_constant + rc->pb_offset;
h->mb.ip_offset = rc->ip_offset + 0.5;

rc->lstep = pow( 2, h->param.rc.i_qp_step / 6.0 );
rc->last_qscale = qp2qscale( 26 + QP_BD_OFFSET );
```

初始的predictor：

```c
static const float pred_coeff_table[3] = { 1.0, 1.0, 1.5 };
for( int i = 0; i < 3; i++ )
{
    rc->last_qscale_for[i] = qp2qscale( ABR_INIT_QP );
    rc->lmin[i] = qp2qscale( h->param.rc.i_qp_min );
    rc->lmax[i] = qp2qscale( h->param.rc.i_qp_max );

    rc->pred[i].coeff_min = pred_coeff_table[i] / 2;
    rc->pred[i].coeff = pred_coeff_table[i];
    rc->pred[i].count = 1.0;
    rc->pred[i].decay = 0.5;
    rc->pred[i].offset = 0.0;

    for( int j = 0; j < 2; j++ )
    {
        rc->row_preds[i][j].coeff_min = .25 / 4;
        rc->row_preds[i][j].coeff = .25;
        rc->row_preds[i][j].count = 1.0;
        rc->row_preds[i][j].decay = 0.5;
        rc->row_preds[i][j].offset = 0.0;
    }
}

rc->pred_b_from_p->coeff_min = 0.5 / 2;
rc->pred_b_from_p->coeff = 0.5;
rc->pred_b_from_p->count = 1.0;
rc->pred_b_from_p->decay = 0.5;
rc->pred_b_from_p->offset = 0.0;
```

初始的ratefactor:

```c++
// 初始的cplxr_sum
rc->cplxr_sum = .01 * pow( 7.0e5, rc->qcompress ) * pow( h->mb.i_mb_count, 0.5 );
// 初始的每帧期望bit数
rc->wanted_bits_window = 1.0 * rc->bitrate / rc->fps;
```

初始的cplxr_sum被定义为：

$rc->cplxr-sum = 0.01\times(7\times10^{5})^{rc->qcompress}\times (h->mb.i\_mb\_count)^{0.5}$

ratefactor被定义为：

$ratefactor = \frac{rc-> wanted\_bits\_window}{rc->cplxr\_sum}$

## 编码前计算帧级QP

计算帧级QP的过程实际就是计算qscale

### 参考帧的qscale计算

1. **初步的计算qscale**

   $recq = {(\frac{BASE\_FRAME\_DURATION}{\frac{frame->i\_duration}{2 \times fps}})}^{1-h->param.rc.f\_qcompress}$

    $qscale = \frac{recq}{ratefactor}$

   其中

   + $BASE\_FRAME\_DURATION$固定为0.04

   + $ratefactor = \frac{rc-> wanted\_bits\_window}{rc->cplxr\_sum}$

2. **第一次调整qscale**

   计算当前已经编完了多少秒的视频：

   ```c
   double time_done =  h->i_frame / rcc->fps;
   ```

   计算截至目前预期的bits：

   ```c
   wanted_bits = time_done * rcc->bitrate;
   ```

   调整qscale(首帧不调整)：

    $overflow=1+\frac{predicted\_bits-wanted\_bits}{MAX(1, \sqrt{time\_done)}}$

    $qscale=old\_qscale \times overflow$

   其中：

   + predicted_bits代表截至目前已经编码完成的帧的实际消耗bits

3. **clip qscale，防止qscale波动太大**

   ```c
   double lmin = rcc->last_qscale_for[pict_type] / rcc->lstep;
   double lmax = rcc->last_qscale_for[pict_type] * rcc->lstep;
   if( overflow > 1.1 && h->i_frame > 3 )
       lmax *= rcc->lstep;
   else if( overflow < 0.9 )
       lmin /= rcc->lstep;
   q = x264_clip3f(q, lmin, lmax);
   ```

   其中：

   + **last_qscale_for**初始值为qp2qscale(ABR_INIT_QP)，在每次初步计算qscale后将其更小到last_qscale_for中
   + lstep代表qscale的的最大变化幅度
   + lmin和lmax表示在上一个qscale的基础上，当前帧的qscale能波动的范围

4. **第二次调整qscale(VBV)**

   先列出此时的VBV相关的几个变量

   ```c
   rc->buffer_fill = 
   ```

   

   1. 首先基于上面得到的qscale，预测当前帧消耗的bits
   2. 

### 非参考B帧的qscale计算

## 宏块级QP调整

## 编码完一帧后码控参数更新

