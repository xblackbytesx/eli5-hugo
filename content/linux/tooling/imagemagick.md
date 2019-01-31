---
title: "Imagemagick"
date: 2019-01-31T11:36:35+01:00
draft: false
---

### Installing Imagemagick
```
sudo pacman -S imagemagick
```

### Stripping metadata (EXIF)
```
mogrify -strip *.JPG
```

### Resizing image by half
```
mogrify -resize 50% *.JPG
```

### Of course you can combin all of these arguments
```
mogrify -resize 50% -strip *.JPG
```
