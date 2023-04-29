# Ok Palette

Still a WIP, but...

Ok Palette takes an image and produces a color palette consisting of the image's average colors.
It does this by converting the image's pixels to the [Oklab](https://bottosson.github.io/posts/oklab/) color space
and then performing k-means clustering.
By using a proper color space for color difference and a more accurate clustering algorithm,
this helps to ensure that the generated palette is truly representative of the input image.

One of the main intended use cases for Ok Palette is to generate colors for a theme based off a wallpaper.
In line with this goal, Ok Palette also supports printing the final average colors in multiple Okhsl lightness levels.
For example, you can specify a low lightness level for background colors
and a high lightness for foreground text in order to achieve a certain contrast ratio.
The [Okhsl](https://bottosson.github.io/posts/colorpicker/) color space is ideal for this,
because as the lightness is changed, Okhsl preserves the hue and saturation of the color
(better than other color spaces like HSL). You can see some of the [examples]() below.

## Notes

Ok Palette uses the [`image`](https://github.com/image-rs/image) crate to load images
and so should work with any image format supported by it.
Although, currently only png, jpeg, and avif have been (somewhat) tested.
Also, due an [issue](https://github.com/image-rs/image/issues/1647) with avif support in the image crate,
avif support is currently handled using `aom`.
So, loading avif images requires having `aom` on your system
(e.g., [Arch package](https://archlinux.org/packages/extra/x86_64/aom/)).

## Performance: ~~blazingly~~ smolderingly fast

Despite using k-means which is more accurate but slower than something like median cut quantization,
Ok Palette still seems to be decently fast. For example, for a 1920x1080 jpeg image on my hardware,
Ok Palette takes about 100-200ms to complete using the default options.
(15-50ms of this time is simply loading the image from disk and decoding it.)
The plan is to reduce this time further by implementing multi-threading
and/or possibly SIMD once [portable-simd](https://github.com/rust-lang/rust/issues/86656) becomes stable.

(Benchmarks will hopefully come soon as well!)

# Examples

Let's use the following photo for the examples below.

![Jewel Changi Aiport Waterfall](doc/Jewel%20Changi.jpg)

Running Ok Palette for this image with the default options gives the following sRGB hex values:

```bash
> ok-palette 'img/Jewel Changi.jpg'
010303 091E22 104852 E7E7E9 BEA8A9 527A8F A76965 583830
```

If your terminal supports true color,
then you can use `-o swatch` to see blocks of the output colors.

```bash
> ok-palette 'img/Jewel Changi.jpg' -o swatch
```

![](doc/swatch1.svg)

We can increase the color accuracy by increasing the number of trials, `-t`, and lowering the convergence threshold, `-e`.

```bash
> ok-palette 'img/Jewel Changi.jpg' -t 4 -e 0.01 -o swatch
```

![](doc/swatch2.svg)

Oh no, that's too accurate!
The image is made up of mostly black, white, and blue-green,
so the other colors are hardly coming through anymore.

We can try increasing k, but now we have a lot of colors, and some are very similar.

```bash
> ok-palette 'img/Jewel Changi.jpg' -k 12 -t 4 -e 0.01 -o swatch
```

![](doc/swatch3.svg)

Alternatively, let's ignore color lightness to merge similar colors together, thereby bringing out other colors in the process:

```bash
> ok-palette 'img/Jewel Changi.jpg' --ignore-lightness -t 4 -e 0.01 -o swatch
```

![](doc/swatch4.svg)

That's better! Unfortunately, now white and black have been merged into a dark gray. We can, however, specify additional lightness levels to print the colors in with `-l`.

```bash
> ok-palette 'img/Jewel Changi.jpg' -l 10,30,50,70 --ignore-lightness -t 4 -e 0.01 -o swatch
```

![](doc/swatch5.svg)

# References

- [kmeans-colors](https://github.com/okaneco/kmeans-colors/) served as the basis for Ok Palette.
  If you want to perform other k-means related operations on images or prefer the CIELAB colorspace, then check it out!
- The work by [Dr. Greg Hamerly and others](https://cs.baylor.edu/~hamerly/software/kmeans)
  was a very helpful resource for the k-means implementation in Ok Palette.
- The awesome [palette](https://github.com/Ogeon/palette) library is used for all color conversions.
