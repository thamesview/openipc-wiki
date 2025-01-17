Ingenic T31 Development Application
===================================

YUV and RAW Data
----------------

### What is YUV data and RAW data?

#### RAW data

RAW data is the most original data from the sensor.

There is no way to use it directly, you have to do ISP, color correction, enhancement, HDR,
interpolation to RGB, conversion to YUV, and finally use it.

#### RGB data

Three primary colors data, red, green and blue data, commonly used data formats are RGB888, true color, RGB565,
RGB555 and other formats, of which RGB888 is true color, occupies 24-bit data, while RGB565 occupies 16-bit data,
most human eyes will not be so sensitive to color, so generally only need to use RGB565 format.

#### YUV data

Although all images can choose the RGB format to represent, RGB images do not support black and white images very well.

Therefore, most of the time we need to convert RGB to YUV format.

__Y__ in the __YUV__ represents the brightness, that is grayscale map, __U__ and __V__ correspond to __Cb__
and __Cr__, respectively, on behalf of the chromaticity, the role of describing the image color as well as
saturation, used to specify the color of the pixel.

#### NV12

The difference between __NV12__ and __NV21__ lies only on the UV plane, both UV but alternation orders are reversed,
__NV12__ is VU alternation. The rest and __NV21__ is the same, is also 4 __Y__ component corresponds to the same
group of UV components; in fact, __YUV:4:2:0__ means 4 __Y__ component data, 2 __U__ and __V__ component data.
The approximate occupied space is 12 bits of data.

![](assets/net-img-72655bea607588644d18d3626ac8c16b-20230919115838-17trt69.png)

# 2: How to get YUV and RAW data for Ingenic T31

## 2.1: Key attribute: set channel attribute to YUV attribute or RAW attribute

RAW data

```
fs_chn_attr[0].pixFmt = PIX_FMT_RAW;
ret = IMP_FrameSource_SetChnAttr(0, &fs_chn_attr[0]);
if (ret < 0) {
    IMP_LOG_ERR(TAG, "%s(%d):IMP_FrameSource_SetChnAttr failed\n", __func__, __LINE__);
    return -1;
}
```

YUV data

```
/* Step.3 Snap raw config */
ret = IMP_FrameSource_GetChnAttr(0, &fs_chn_attr[0]);
if (ret < 0) {
    IMP_LOG_ERR(TAG, "%s(%d):IMP_FrameSource_GetChnAttr failed\n", __func__, __LINE__);
    return -1;
}
fs_chn_attr[0].pixFmt = PIX_FMT_NV12;//PIX_FMT_YUYV422;
ret = IMP_FrameSource_SetChnAttr(0, &fs_chn_attr[0]);
if (ret < 0) {
    IMP_LOG_ERR(TAG, "%s(%d):IMP_FrameSource_SetChnAttr failed\n", __func__, __LINE__);
    return -1;
}

/* Step.3 config sensor reg to output colrbar raw data*/
/* to do */

/* Step.4 Stream On */
if (chn[0].enable){
    ret = IMP_FrameSource_EnableChn(chn[0].index);
    if (ret < 0) {
        IMP_LOG_ERR(TAG, "IMP_FrameSource_EnableChn(%d) error: %d\n", ret, chn[0].index);
        return -1;
    }
}
```

#### Get YUV image data & RAW data

```
int m = 0;
for (m=1;m<=51;m++) {
    ret = IMP_FrameSource_GetFrame(0, &frame_bak);
    if (ret < 0) {
        IMP_LOG_ERR(TAG, "%s(%d):IMP_FrameSource_GetFrame failed\n", __func__, __LINE__);
        return -1;
    }
    if(m%50==0) {
        fwrite((void *)frame_bak->virAddr, frame_bak->size, 1, fp);
        fclose(fp);
    }
    IMP_FrameSource_ReleaseFrame(0, frame_bak);
    if (ret < 0) {
        IMP_LOG_ERR(TAG, "%s(%d):IMP_FrameSource_ReleaseFrame failed\n", __func__, __LINE__);
        return -1;
    }
}
```

#### Release of resources

It is the same as the original counter-initialization and will not be repeated.


### Experimental Phenomena

![](assets/net-img-15097e3079df1d2f942210a0d01c12b1-20230919115839-c12da3z.png)

RAW data is very large without compression, much larger than JPEG, so we generally use JPEG format
for network transmission to save bandwidth.

Only when the image quality is not up to standard, we will save the RAW data and YUV data to analyze
the image quality problem.

Size comparison. Generally a five megapixel photo:
- JPEG: 105KB
- YUV: 7200KB
- RAW: 9600KB

Below is a picture of what the YUV image actually looks like.

![](assets/net-img-2d98db182030b8737b8d380492192294-20230919115839-al19eq9.png)

Appendix: screenshot YUV full code:

```
/*
 * sample-snap-raw.c
 *
 * Copyright (C) 2018 Ingenic Semiconductor Co.,Ltd
 */

#include <imp/imp_log.h>
#include <imp/imp_common.h>
#include <imp/imp_system.h>
#include <imp/imp_framesource.h>
#include <imp/imp_encoder.h>

#include "sample-common.h"

#define TAG "Sample-Snap-RAW"
extern struct chn_conf chn[];

int main(int argc, char *argv[])
{
    int ret;

    IMPFrameInfo *frame_bak;
    IMPFSChnAttr fs_chn_attr[2];
    FILE *fp;

    fp = fopen("/tmp/snap.yuv", "wb");
    if(fp == NULL) {
        IMP_LOG_ERR(TAG, "%s(%d):open error !\n", __func__, __LINE__);
        return -1;
    }

    /* Step.1 System init */
    ret = sample_system_init();
    if (ret < 0) {
        IMP_LOG_ERR(TAG, "IMP_System_Init() failed\n");
        return -1;
    }

    /* Step.2 FrameSource init */
    if (chn[0].enable) {
        ret = IMP_FrameSource_CreateChn(chn[0].index, &chn[0].fs_chn_attr);
        if(ret < 0){
            IMP_LOG_ERR(TAG, "IMP_FrameSource_CreateChn(chn%d) error !\n", chn[0].index);
            return -1;
        }

        ret = IMP_FrameSource_SetChnAttr(chn[0].index, &chn[0].fs_chn_attr);
        if (ret < 0) {
            IMP_LOG_ERR(TAG, "IMP_FrameSource_SetChnAttr(chn%d) error !\n",  chn[0].index);
            return -1;
        }
    }

    /* Step.3 Snap raw config */
    ret = IMP_FrameSource_GetChnAttr(0, &fs_chn_attr[0]);
    if (ret < 0) {
        IMP_LOG_ERR(TAG, "%s(%d):IMP_FrameSource_GetChnAttr failed\n", __func__, __LINE__);
        return -1;
    }
#if 0
    ret = IMP_ISP_Tuning_SetISPBypass(IMPISP_TUNING_OPS_MODE_DISABLE);
    if (ret < 0) {
        IMP_LOG_ERR(TAG, "%s(%d):IMP_ISP_Tuning_SetISPBpass failed\n", __func__, __LINE__);
        return -1;
    }
#endif
    fs_chn_attr[0].pixFmt = PIX_FMT_NV12;//PIX_FMT_YUYV422;
    ret = IMP_FrameSource_SetChnAttr(0, &fs_chn_attr[0]);
    if (ret < 0) {
        IMP_LOG_ERR(TAG, "%s(%d):IMP_FrameSource_SetChnAttr failed\n", __func__, __LINE__);
        return -1;
    }

    /* Step.3 config sensor reg to output colrbar raw data*/
    /* to do */

    /* Step.4 Stream On */
    if (chn[0].enable){
        ret = IMP_FrameSource_EnableChn(chn[0].index);
        if (ret < 0) {
            IMP_LOG_ERR(TAG, "IMP_FrameSource_EnableChn(%d) error: %d\n", ret, chn[0].index);
            return -1;
        }
    }

    /* Step.4 Snap raw */
    ret = IMP_FrameSource_SetFrameDepth(0, 1);
    if (ret < 0) {
        IMP_LOG_ERR(TAG, "%s(%d):IMP_FrameSource_SetFrameDepth failed\n", __func__, __LINE__);
        return -1;
    }

    int m = 0;

    for (m=1;m<=51;m++) {
        ret = IMP_FrameSource_GetFrame(0, &frame_bak);
        if (ret < 0) {
            IMP_LOG_ERR(TAG, "%s(%d):IMP_FrameSource_GetFrame failed\n", __func__, __LINE__);
            return -1;
        }
        if(m%50==0) {
            fwrite((void *)frame_bak->virAddr, frame_bak->size, 1, fp);
            fclose(fp);
        }
        IMP_FrameSource_ReleaseFrame(0, frame_bak);
        if (ret < 0) {
            IMP_LOG_ERR(TAG, "%s(%d):IMP_FrameSource_ReleaseFrame failed\n", __func__, __LINE__);
            return -1;
        }
    }
    ret = IMP_FrameSource_SetFrameDepth(0, 0);
    if (ret < 0) {
        IMP_LOG_ERR(TAG, "%s(%d):IMP_FrameSource_SetFrameDepth failed\n", __func__, __LINE__);
        return -1;
    }
    /* end */

#if 0
    ret = IMP_ISP_Tuning_SetISPBypass(IMPISP_TUNING_OPS_MODE_ENABLE);
    if (ret < 0) {
        IMP_LOG_ERR(TAG, "error:(%s,%d),IMP_ISP_Tuning_SetISPBypass failed.\n",__func__,__LINE__);
        return -1;

    }
#endif
    /* Step.5 Stream Off */
    if (chn[0].enable){
        ret = IMP_FrameSource_DisableChn(chn[0].index);
        if (ret < 0) {
            IMP_LOG_ERR(TAG, "IMP_FrameSource_DisableChn(%d) error: %d\n", ret, chn[0].index);
            return -1;
        }
    }

    /* Step.6 FrameSource exit */
    if (chn[0].enable) {
        /*Destroy channel i*/
        ret = IMP_FrameSource_DestroyChn(0);
        if (ret < 0) {
            IMP_LOG_ERR(TAG, "IMP_FrameSource_DestroyChn() error: %d\n", ret);
            return -1;
        }
    }

    /* Step.7 recover sensor reg to output normal image*/
    /* to do */

    /* Step.8 System exit */
    ret = sample_system_exit();
    if (ret < 0) {
        IMP_LOG_ERR(TAG, "sample_system_exit() failed\n");
        return -1;
    }

    return 0;
}
```

[toc]: index.md
