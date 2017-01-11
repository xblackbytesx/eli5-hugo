+++
draft = false
date = "2017-01-11T15:27:34+01:00"
title = "pacman"
categories = ["Linux","Tooling"]

+++

## Installing a package
```
sudo pacman -S <package>
```

## Searching for a package
```
sudo pacman -Ss <description/name>
```

## Removing packages

To remove a single package, leaving all of its dependencies installed:
```
sudo pacman -R <package>
```

To remove a package and its dependencies which are not required by any other installed package:
```
sudo pacman -Rs <package>
```

## Cleaning the package cache
The built-in option to remove all the cached packages that are not currently installed is:
```
sudo pacman -Sc
```

To clean the entire cache run this (be careful with this one):
```
sudo pacman -Scc
```
