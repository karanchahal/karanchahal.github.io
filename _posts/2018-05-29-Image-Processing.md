This is an excerpt from the wiki of the __object detection software__ I'm currently implementing [here](https://github.com/karanchahal/object-detection/wiki). 
We are building a free and open state of the art object detector for production environments. Join in if you would like to contribute !

In this article we are going to be talking about how we process images to be input into our deep learning model. There are a lot of things to be kept in mind.
We want to devise a solution that is
```
1. Portable
2. Fast
3. Supported
4. Powerful
```
# Choice Of Library

The software will use __Open CV__ for all of it's image transformations. One of the central goals of the project is to be __fast__. 

Across the internet it has been established that __OpenCV__ is the _fastest_ tool for dealing with images. Other python libraries like _PIL_, _sk-image_ are good to get something done quick, but aren't as good for production systems. Open CV has API's in _C++, Python and a lot of languages_. This makes it a ideal fit to when we want to port our framework.

Open CV consistently gives a __10-100x__ boost in performance compared to these other libraries.

The one _disadvantage_ of using OpenCV is it has really _bad/obscure documentation_ for even the simplest things. To make this process simpler for the newcomer, the page shall keep _track of all transformations done with OpenCV in this project_.

For those who're looking for comparisons between opencv and other libraries , [this](https://www.kaggle.com/vfdev5/pil-vs-opencv) might come in handy. read more about OpenCV [here](https://opencv.org/)


# Core Beliefs For Our Images

Here, we list simple transformations done using OpenCV in the Python API. We keep a few core principles when interacting with images.

1. The images should be __numpy__ arrays
2. The images should be in __RGB__ format.
3. Images should be __normalised__ between 0 and 1. That is we divide the image pixels by 255.
4. Images of type numpy will have the shape __(height,width,channels)__ . (Batch size to be decided later)

These principles are very __important__ as we won't have deal with confusing bugs later on if we follow them strictly.

# Reading an Image

We read in an image using the standard OpenCV function

```python
flags = cv2.IMREAD_UNCHANGED+cv2.IMREAD_ANYDEPTH+cv2.IMREAD_ANYCOLOR # flags for images
img = cv2.imread('path-to-file.jpg',flags)
```

We read in an image and then normalise it by dividing the pixels by 255. We also convert them into float32 values.
```python
im = cv2.imread(str(path), flags).astype(np.float32)/255
```

OpenCV always returns an image in a Numpy array format and in the __BGR__ format. This is especially important, all other libraries expect the image to be in __RBG__ format. Hence, we always convert images read to the RGB format for uniformity.

```python
cv2.cvtColor(im, cv2.COLOR_BGR2RGB)
```

So to sum it up, after reading an image , we always get a __float32 numpy array__

# Resizing the Image

Resizing the image is one of the most important transformations in Computer Vision algorithms. 

## Objective

``` Resize an image such that the smallest side is of some side x, while preserving the aspect ratio```

We can then input these resized images into our model.

Some questions on the validity of this approach are:

### What about the images that are too small to be resized to this size ?

The images could turn out to be extremely blurred / choppy and won't result in good predictions. 

A __solution__ to this could be 

1. Allow sides of input images to be greater than certain values, eg: smallest side should be larger than 200 pixels.

### Guiding Statement

The computer vision module should be able to consume images of varying sizes. If an image is of high quality, we don't want to compress that image and loose all of the detail.

Generally our computer vision algorithms should have the capability to recognise images at various scales. This is possible with the FPN network  described [here](https://arxiv.org/abs/1612.03144).

Hence, a mechanism to input images of scales ranging from __224-1000 pixels__ could be a good solution. By _scales_, we mean the shortest side of the image. This criteria leads to a change of how our deep learning models will be architected but that is a problem for an other time. For those who are interested, look up __Global Average Softmax__.


Anyway back to the question on resizing the image.

We need to construct a function that resizes the image such that the length of the shortest side should be x. OpenCV provides a nice function for just that.

```python
cv2.resize(image,(target_width,target_height),interpolation=interpolation)
```

To preserve aspect ratio, we find out the target height or width for some size .

Eg: If the size we want to resize down to is 224. We calculate aspect ratio before resizing

```
ratio = height/width
```

And then we multiply/divide the longer side by this ratio depending on whether the longer side is height/width.

In this example, our shorter side is the width. Hence our resizing operation will be:
```python
ratio = float(h/w)
target_height = int(ratio*size)
return cv2.resize(image,(size,target_height), interpolation=interpolation)
```

## Side Effect of Resizing

Since we're building an object detection framework. Our training data has 2 objects.

1. Image
2. Bounding Box Coordinates

Hence, when we actually perform the resizing of the image to a fixed size, we need to __modify our bounding box coordinates__ too ! If not the bounding box coordinates would not be compatible with the resized image and our network may get confused.

So our training pipeline so far will look like something like this.

```python
for image in dataset:
    h,w = image.size
    image = resize(image,224)
    new_h,new_w = image.size
    
    width_offsets, height_offsets = calculateOffsets(h,w,new_h,new_w)
    
    # make prediction
    x,y,width,height = ObjectDetectionModel(image) # assuming image has only one object in it
    
    x = x*width_offset
    y = y*height_offset
    width = width*width_offset
    height = height*height_offset

```

Hence, we modify the bounding box coordinates that we get through prediction to fit the original image's dimensions.

# Conclusion

Today, we learnt how to

1. Read in an Image
2. Resize the image
3. Take care of bounding boxes




