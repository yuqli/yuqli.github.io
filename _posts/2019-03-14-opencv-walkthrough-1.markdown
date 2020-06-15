---
layout: post
title:  "OpenCV dnn module walkthrough 1: background, important files and classes"
date:   2019-03-14 22:53:34 -0400
categories: jekyll update
---
(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

With the help of CLion documentation, I was able to find the base class for layers in OpenCV dnn. Here you go, check line 174. 

This class has the following members and member functions, and we'll discuss them one-by-one...

# members

- `std::vector<Mat> blobs`: to store a list of computed parameters. Will be read by Net::getParam()
- `String name`: name of the layer instance
- `String type`: type name used for creating layer from layer factory
- `int preferableTarget`: cpu or gpu or other platform? Used for layer forwarding.

# layer computation

- various finalize(const std::vector<Mat*> &input, std::vector<Mat> &output) functions and various forward functions : used to compute one forward pass given a input data
- `void run()` : used to allocate layers and compute output. Deprecated.
- `virtual int inputNameToIndex(String inputname)`: given an inputname, return the index of input blob from the blobs. This function further reveals how parameters are stored in the Layer class: with an index and name. Each layer input and output can be labeled to easily identify them using "layer name". For example, an implementation occurs at opencv/modules/dnn/src/layers/recurrent_layers.cpp, where for LSTM layer if inputname == 'x' then return 0, else return -1
- `virtual int outputNameToIndex(String outputname)` : this is similar to inputNameToIndex

# utilities

- `virtual bool supportBackend (int backendId)`: ask a layer if a particular backend is supported.
- `virtual Ptr<BackendNode> initHalide( const std::vector<Ptr<BackendWrapper> > &inputs)`: this function seems to return a Halide backend node. But I'm not very familiar with the Halide language so I'll skip it for now. But do note the unfamiliar data structure BackendWrapper and BackendNode
- `virtual Ptr<BackendNode> initInfEngine()`: seems to connect to Halide and thus skip
- `virtual void applyHalideScheduler()`: skip for the same reason as above

# computational graph

- `virtual Ptr<BackendNode> tryAttach(const Ptr<BackendNode> & node)`: this function relates tightly to the implementation of BackendNote class. It checks if layer attached successfully and returns non-empty cv::Ptr to node of the same backend. A concrete implementation lies at opencv/modules/dnn/src/dnn.cpp, in which it simply returns an empty Ptr<BackendNote>()
- `virtual bool setActivation(const Ptr<ActivationLayer>& layer)`. The documentation says it will return true if the activation layer has been attached successfully, but in its implementation at `opencv/modules/dnn/src/dnn.cpp`, the function seems to always return false. Note the new data structure ActivationLayer here.
- `virtual bool tryFuse(Ptr<Layer>& top)`: try to fuse current layer with the next one. top is the top next layer to be fused. Returns true if fusion was performed. But in its implementation the function always return false.
- `virtual void getScaleShift(Mat & scale, Mat & shift) const`: modifies layer parameters by a scale and a shift factor. By default this function is implemented as an empty scale and an empty shift parameter, thus no scale and no shift
- `virtual void unsetAttached()` : as the name suggests

# resources

- `virtual void getMemoryShapes()` : Not sure what it does.
- `virtual int64 getFLOPS()`: guess it returns FLOPS

# big five:

- `Layer()`: empty constructor
- `explicit Layer(const LayerParams &params)`: copy constructor
- `void setParamesFrom(const LayerParams &params)`: setter
- `virtual ~Layer()`: virtual destructor
 