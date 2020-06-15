---
layout: post
title:  "OpenCV dnn module walkthrough 2: layers – the base class"
date:   2019-03-19 22:33:34 -0400
categories: jekyll update
---
(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

This blog aims to briefly present a roadmap to understand the source code of OpenCV's `dnn` module.

# Example usage
Before diving deep in to the code, you can find an example usage of this module here. This example shows how to load a pretrained Caffe model into OpenCV Net and compute a forward pass on test data. This is how to load a Caffe model into OpenCV:

{% highlight ruby %}
CV_Assert(parser.has("model"));
Net net = readNet(model, config, framework);
net.setPreferableBackend(backendId);
net.setPreferableTarget(targetId);
{% endhighlight %}

... and here is how to compute a forward pass and get classification results on test data:

{% highlight ruby %}
blobFromImage(frame, blob, scale, Size(inpWidth, inpHeight), mean, swapRB, false);
net.setInput(blob);
Mat prob = net.forward();
Point classIdPoint;
double confidence;
minMaxLoc(prob.reshape(1, 1), 0, confidence, 0, classIdPoint);
int classId = classIdPoint.x;
{% endhighlight %}

So the key object in the above snippets is Net. It is initialized by the `readNet` function, and has various methods, including a `forward` that is very much like other deep learning frameworks. Note it's mostly a wrapper for loading pre-trained models, and does not support model training by itself. As the documentation  says, it basically allows to create and manipulate comprehensive artificial neural networks.

A note on why OpenCV framework is only responsible for inference and not training - quoting Intel Distribution of OpenVino Tookit :

> Training deep learning networks is typically performed in data centers or server farms and the inference often take place on embedded platforms that are optimized for performance and power consumption. These platforms are typically limited from the software perspective:
> - programming languages
> - third party dependencies
> - memory consumption
> - supported operating systems
> - and the platforms are limited from the hardware perspective:
> - different data types
> - limited power envelope
>
> Because of these limitations, it is usually not recommended, and sometimes not possible, to use original training framework for inference. As an alternative, use dedicated inference APIs that are optimized for specific hardware platforms.
>
> As a result, Intel also offer an Intermediate Representation (IR) of the model based on the trained network topology, weights, and bias values, which will be an `.xml` file. We will later see the `Net` class in OpenCV is also able to load a model from this file format.

# The "Net" class
To understanding how `Net` is implemented, the next questions are 
- 1) how it links to PyTorch models and 
- 2) how it computes forward pass and 3) how to add a GPU backend to it? 

Before answering these questions, it might help to have a glimpse of what's inside the `dnn` module source code. Here is a link:

- Folders named "caffe", "torch", "tensorflow", "darknet", "onnx": these are probably functions specific to one of the neural network framework backends supported by OpenCV, as can be found in the Wiki.
- A folder named "Layers", seems to consist of implementations of building block layer functions. e.g. the file "convolution_layer.cpp" consists of three classes: a base class for convolution, a derived convolution class, and a deconvolution layer.
- A folder named "ocl4dnn". Have no clue.
- A folder named "opencl" seems to have lots of opencl code
- a folder "vkcom" seems to have code related to vulkan as its graphics computing engine?
- the `dnn.cpp` file we are interested in.
- various other cpp files.

Next we'll explore the `Net` class!  It's declared in the `dnn.hpp` file and defined in the `dnn.cpp` file (3809 line of code). Use a search function to find definition of Net... I used CLion to quickly find it and jump between declarations and definitions. This class consists of the following groups of functions (I omit function signatures for simplicity).

# a) constructors and destructors.
Names are straightforward to understand what these functions do.

- Net() constructor
- ~Net() destructor
- readFromModelOptimizer() : load an Intel Intermediate Representation file for deep learning models, format is xml file
- empty()

# b) layer operations
- addLayer()
- addLayerToPrev()
- getLayerID()
- getLayerNames()
- LayerID
- getLayer()
- connect()
- connect()
- setInputNames()
- forward()

To really understand these functions, it seems necessary for us to understand the layer class. We'll do that in the next section.

# c) getters and setters
Again rather straightforward.

- setHalideScheduler(): Halide is a language for data-parallel computations.
- setPreferableBackend(): takes int backendID as input. Acceptable backend engines are introduced in the last section...
- setPreferableTarget(): this function specifies which device as target. takes int device ID as input. example devices include DNN_TARGET_CPU, DNN_TARGET_OPENCL, DNN_TARGET_OPENCL_FP16, DNN_TARGET_MYRIAD, DNN_TARGET_FPGA
- setInput() : set the input value for the network
- setParam(): set new values for parameter layers. Worthy of discussing more after knowing how `layers` are constructed.
- getParam()
- getUnconnectedOutLayers(): said to "return indexes of layers with unconnected outputs". Not sure what that means though.
- getUnconnectedOutLayerNames()
- getLayersShapes(): with a few overloading functions with different function signatures. Returns ALL layer shapes and store them in a MatShape or vector<MatShape>
- getLayerShapes(): Return a single layer shape.
- getFLOPs(): compute FLOPs for a specific model.
- getLayerTypes()
- getLayersCounts()
- getMemoryConsumption(): with a few overloading variations. Return the number of bytes for storing all weights and data
- enableFusion(): enable or disable layer fusion in the network
- getPerfProfile(): returns profiling results.

# d) member
Apart from methods, `Net` also have a member Impl defined as follows:

{% highlight ruby %}
struct Net::Impl
{
    typedef std::map<int, LayerShapes> LayersShapesMap;
    typedef std::map<int, LayerData> MapIdToLayerData;

    Impl()
    {
        //allocate fake net input layer
        netInputLayer = Ptr<DataLayer>(new DataLayer());
        LayerData &inpl = layers.insert( make_pair(0, LayerData()) ).first->second;
        inpl.id = 0;
        netInputLayer->name = inpl.name = "_input"
        inpl.type = "__NetInputLayer__"
        inpl.layerInstance = netInputLayer;
        layerNameToId.insert(std::make_pair(inpl.name, inpl.id));

        lastLayerId = 0;
        netWasAllocated = false;
        fusion = true;
        preferableBackend = DNN_BACKEND_DEFAULT;
        preferableTarget = DNN_TARGET_CPU;
        skipInfEngineInit = false;
    }

    Ptr<DataLayer> netInputLayer;
    std::vector<LayerPin> blobsToKeep;
    MapIdToLayerData layers;
    std::map<String, int> layerNameToId;
    BlobManager blobManager;
    int preferableBackend;
    int preferableTarget;
    String halideConfigFile;
    bool skipInfEngineInit;
    // Map host data to backend specific wrapper.
    std::map<void*, Ptr<BackendWrapper> > backendWrappers;

    int lastLayerId;

    bool netWasAllocated;
    bool fusion;
    std::vector<int64> layersTimings;
    Mat output_blob;

    Ptr<BackendWrapper> wrap(Mat& host)
    {
        if (preferableBackend == DNN_BACKEND_OPENCV && preferableTarget == DNN_TARGET_CPU)
            return Ptr<BackendWrapper>();

        MatShape shape(host.dims);
        for (int i = 0; i < host.dims; ++i)
            shape[i] = host.size[i];

        void* data = host.data;
        if (backendWrappers.find(data) != backendWrappers.end())
        {
            Ptr<BackendWrapper> baseBuffer = backendWrappers[data];
            if (preferableBackend == DNN_BACKEND_OPENCV)
            {
                CV_Assert(IS_DNN_OPENCL_TARGET(preferableTarget));
                return OpenCLBackendWrapper::create(baseBuffer, host);
            }
            else if (preferableBackend == DNN_BACKEND_HALIDE)
            {
                CV_Assert(haveHalide());
  #ifdef HAVE_HALIDE
                return Ptr<BackendWrapper>(new HalideBackendWrapper(baseBuffer, shape));
  #endif  // HAVE_HALIDE
            }
            else if (preferableBackend == DNN_BACKEND_INFERENCE_ENGINE)
            {
                return wrapMat(preferableBackend, preferableTarget, host);
            }
            else
                CV_Error(Error::StsNotImplemented, "Unknown backend identifier");
        }

        Ptr<BackendWrapper> wrapper = wrapMat(preferableBackend, preferableTarget, host);
        backendWrappers[data] = wrapper;
        return wrapper;
    }
{% endhighlight %}

It's probably not the best idea posting the code here... But we can see really this "implementation" is trying to set up configurations and perhaps populate layers. So at the next step we'll have a look at the `Layer` class and its operations.