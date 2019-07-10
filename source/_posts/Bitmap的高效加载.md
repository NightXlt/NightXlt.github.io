title:  Bitmap的高效加载
date: 2019-2-23
tags: [Android]
categories: Android开发艺术探索
description: 怎么又OOM啦
---
## 降低加载Bitmap的内存
一言以蔽之：加载Bitmap前将Bitmap压缩至ImageView大小。
实现：利用BitmapFactory.Options加载所需尺寸图片。通过BitmapFactory.Options来缩放图片，主要利用了他的inSampleSize参数，即采用率。当inSampleSize为1时，采样后的图片大小为原始图片的原始大小；当imSampleSize大于1时，比如为2，那么采样后的图片其宽/高均为原图大小的1/2，而像素为原图的1/4，其占有的内存大小也为原图的1/4.拿一张1024 * 1024像素的图片来说，假定采用ARGB8888格式存储，那么他占有的内存为1024*1024*4，即4MB，如果imSampleSIze为2，那么采样后的图片其内存占用只有512* 512 * 4，即1MB。可以发现采样率imSampleSize必须是大于1的整数才会有缩小的效果，并且采样率同时作用于宽、高，这导致缩放后的图片大小以采用率的2次方递减，即缩放比例为1/(imSamolSize的2次方)，比如inSamoleSize为4，那么缩放比例就是1/16。

获取采样率遵循如下流程：

1）将BitmapFactoryFactory.Optiobs的inJustDecodeBounds参数设为true并加载图片。（为了获取图片宽高）

2）从BitmapFactory.Options中取出图片的原始宽高信息，他们对应于outWidth和outHeight参数。

3）根据采样率的规则并结合目标View的所需大小计算出采样率inSampleSize.

4）将BirmipFactory.Options的inJustDecodeBounds参数设为false，然后重新加载图片。

```java
   public Bitmap decodeSampledBitmapFromResource(Resources res,
            int resId, int reqWidth, int reqHeight) {
        // First decode with inJustDecodeBounds=true to check dimensions
        final BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeResource(res, resId, options);

        // Calculate inSampleSize
        options.inSampleSize = calculateInSampleSize(options, reqWidth,
                reqHeight);

        // Decode bitmap with inSampleSize set
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeResource(res, resId, options);
    }

    public int calculateInSampleSize(BitmapFactory.Options options,
            int reqWidth, int reqHeight) {
        if (reqWidth == 0 || reqHeight == 0) {
            return 1;
        }

        // Raw height and width of image
        final int height = options.outHeight;
        final int width = options.outWidth;
        Log.d(TAG, "origin, w= " + width + " h=" + height);
        int inSampleSize = 1;

        if (height > reqHeight || width > reqWidth) {
            final int halfHeight = height / 2;
            final int halfWidth = width / 2;

            // Calculate the largest inSampleSize value that is a power of 2 and
            // keeps both
            // height and width larger than the requested height and width.
            while ((halfHeight / inSampleSize) >= reqHeight
                    && (halfWidth / inSampleSize) >= reqWidth) {
                inSampleSize *= 2;
            }
        }

        Log.d(TAG, "sampleSize:" + inSampleSize);
        return inSampleSize;
    }

```