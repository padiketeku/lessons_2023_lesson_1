/**** Start of imports. If edited, may not auto-convert in the playground. ****/
var geometry = /* color: #00ffff */ee.Geometry.Point([146.93508606811457, -36.08312045367881]);
/***** End of imports. If edited, may not auto-convert in the playground. *****/
/* ACKNOWLEDGEMENT
Google Earth Engine Developers
Google Earth Engine Team
*/
/*
Please be sure you attempt this practical using the practical manual. The practical manual gives thorough explanations
to the concepts and scripts. Using the practical manual, the commands, arguments, or parameters used in the scripts will
make more sense to you.Also, you will test your understanding of the remote sensing principles as you work through challenge tasks
and revision questions.
*/
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
/*Prac 4: Spatial filtering (low pass and high pass filters) 
In this prac, we will visually enhance a Sentinel-2 image using a low-pass filter  and
a high-pass filter. You will use Sentinel-2 bottom of atmosphere product image. At the end of the prac, you 
should be able to:
1, retrieve and visualise Sentinel-2 images
2, apply low-pass and high-pass filtering to an image
3, describe the difference between low-pass filtered image and high-pass filtered image
*/

//let's load our working image data, which includes Albury our study area

//given we now know the file id based on lesson 1 work; let's
//load this working image

//create an image variable from the image collection and name this variable 'sen2BOA_Albury'
//var L8Albury=ee.Image('LANDSAT/LC08/C01/T1_SR/LC08_091085_20180117');
var sen2BOA_Albury = ee.Image(ee.ImageCollection('COPERNICUS/S2_SR') 
    .filterDate('2020-04-01','2021-10-01') 
    .filterBounds(ee.Geometry.Point (146.94,-36.08)) 
    .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 10))
    .first ());

// print the image data
print (sen2BOA_Albury, 'Original image data');



var visualization = {
  min: 0,
  max: 10000,
  bands: ['B4', 'B3', 'B2']
};

// set the point location of Albury
Map.setCenter(146.94,-36.08, 8);

Map.addLayer(sen2BOA_Albury, visualization, 'RGB image');

/*
Spatial filtering. This requires a single-band image
*/

var sen2BOA_Albury_nir = sen2BOA_Albury.select(['B8']);

print (sen2BOA_Albury_nir);

// let's add the band 8 (NIR) of this image to the base map
Map.addLayer(sen2BOA_Albury_nir, {bands:['B8'], min:0, max:10000},'Original Image');

//Low pass convolution tends to blur the image

//create a list of weight
var list1=[1,1,1,1,1];
print(list1);

//create 5x5 kernel weights as a 2-D matrix
var list2=[list1,list1,list1,list1,list1];

//create the kernel from the weights
var lowpass_kernel=ee.Kernel.fixed(5,5,list2,-1,-1, true);
print(lowpass_kernel);

// let's smooth the images using the low-pass kernel created
var lowpassImage = sen2BOA_Albury_nir.convolve(lowpass_kernel);

//add output image to the interactive map
Map.addLayer(lowpassImage, {bands:['B8'], min:0, max:10000}, 'Smooth Band 8');

/* high pass filtering - aka edge detection filtering
*/

//create a list of weight
var list3=[-1,-1,-1,-1,-1]; 

//set centre of kernel to 24
var list4=[-1,-1,24,-1,-1]; 
print(list4);

//create 5x5 kernel weights as a 2-D matrix
var list5=[list3,list3,list4,list3,list3];
print(list5);

//create the high-pass kernel from the weights
var highpass_kernel=ee.Kernel.fixed(5,5,list5,-1,-1, false);
print(highpass_kernel);

// sharpen feature edges using the high-pass kernel created

var highpass_edges = sen2BOA_Albury_nir.convolve(highpass_kernel);

// visualise output image 
Map.addLayer(highpass_edges, {bands:['B8'], min:0, max:10000}, 'High-pass Band 5');

/* compare the original image to the filtered images. What is the difference between the two filtered images
*/
