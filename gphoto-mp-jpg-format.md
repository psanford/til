# Gphoto MP.jpg file format notes

The Google Camera app can shot camera stills with a small embedded video (mp4) within it.

To extract the mp4 file get the offset (from the end of the file):

```
$ exiv2 -p x PXL_20210706_000407932.MP.jpg
Xmp.GCamera.MotionPhoto                      XmpText     1  1
Xmp.GCamera.MotionPhotoVersion               XmpText     1  1
Xmp.GCamera.MotionPhotoPresentationTimestampUs XmpText     6  775483
Xmp.xmpNote.HasExtendedXMP                   XmpText    32  E36F0BA32655AE28691954FFF1998B09
Xmp.Container.Directory                      XmpText     0  type="Seq"
Xmp.Container.Directory[1]                   XmpText     0  type="Struct"
Xmp.Container.Directory[1]/Container:Item    XmpText     0  type="Struct"
Xmp.Container.Directory[1]/Container:Item/Item:Mime XmpText    10  image/jpeg
Xmp.Container.Directory[1]/Container:Item/Item:Semantic XmpText     7  Primary
Xmp.Container.Directory[1]/Container:Item/Item:Length XmpText     1  0
Xmp.Container.Directory[1]/Container:Item/Item:Padding XmpText     1  0
Xmp.Container.Directory[2]                   XmpText     0  type="Struct"
Xmp.Container.Directory[2]/Container:Item    XmpText     0  type="Struct"
Xmp.Container.Directory[2]/Container:Item/Item:Mime XmpText     9  video/mp4
Xmp.Container.Directory[2]/Container:Item/Item:Semantic XmpText    11  MotionPhoto
Xmp.Container.Directory[2]/Container:Item/Item:Length XmpText     7  5260251
Xmp.Container.Directory[2]/Container:Item/Item:Padding XmpText     1  0


$ dd if=PXL_20210706_000407932.MP.jpg of=out.mp4 bs=$(($(wc -c PXL_20210706_000407932.MP.jpg  | cut -d' ' -f1) - 5260251)) skip=1
```

Source: https://timojyrinki.gitlab.io/hugo/post/2021-03-30-pixel-motionphoto-microvideo-file-formats/
