---
layout: page
permalink: /lab-software/
title: Lab Software
description: "Rundown on lab software."
tags: [Jekyll, theme, responsive]
---

<section id="table-of-contents" class="toc">
  <header>
    <h3 >Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

Summary of lab and other related software.  If you don't have experience with medical image analysis, check out this page. 

## ANTs
[ANTs](http://stnava.github.io/ANTs/) is most well known for image registration, but it also includes image segmentation (Atropos), bias correction (N4), cortical thickness calculator (DiReCT), and access to a variety of ITK-based image processing tools via ImageMath. If you don't have experience with ANTs, read the scripts in the `Scripts/` directory, as the usage examples contain the best parameter settings for most cases. 

## ITK-SNAP
[ITK-SNAP](http://www.itksnap.org/pmwiki/pmwiki.php?n=Main.HomePage) is used for display and interactive segmentation of medical images.  It also shows overlays and segmentations, among many other features. 

## ANTsR
[ANTsR](http://stnava.github.io/ANTsR/) provides an interface to ANTs from the [R programming language](http://cran.us.r-project.org/).  It makes working with medical images in an interactive way much easier and faster. 

## c3d 
[c3d](http://www.itksnap.org/pmwiki/pmwiki.php?n=Convert3D.Documentation), distributed with ITK-SNAP, provides access to a variety of image processing operations. 

## Pipedream 
[Pipedream](http://sourceforge.net/projects/neuropipedream/) is a system for doing documented, reproducible, standardized analyses of imaging data.  It includes easy-to-use utilities for converting DICOMs to Niftis and organizing data. 
