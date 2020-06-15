---
layout: post
title:  "Using ROS with Conda"
date:   2018-11-09 11:05:34 -0400
categories: jekyll update
---
(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

For the past two weeks I've been playing around with ROS. All was fine until I tried to embed tensorflow and "deep learning" stuff into ROS with conda.

Things went south immediately. Pipelines broke down, what used to work no longer worked anymore. roslaunch showed `not a package` error, import in Python showed package not found even though I just pip installed it...

This post aims to clarify what happens, why my system became a mess, and how to resolve them.

The first problem with error importing modules in python is because ROS conflicts with Conda. This is because ROS sneakly changes the Python path in `~/.bash_rc` file. According to this post by Udacity NanoDegree github:

The problem here is that Python is looking for a package in /opt/ros/kinetic/lib/python2.7/dist-packages/ when it should be looking in the Anaconda installation directories. The reason for this is that with your ROS install, your .bashrc file contains the line source /opt/ros/kinetic/setup.bash, which sets your PYTHONPATH variable to the /opt/ros/kinetic/... location.

The solution is to unset your PYTHONPATH variable in your ~/.bash_rc file, right after sourcing ROS, but before sourcing your Anaconda environment. Use this command unset PYTHONPATH.

The second issue is I used many package installers to install python packages (and possibly python itself). Be careful of online tutorials my dude! I ends up having tens of python paths, scattered around different virtual environments, and things became very convoluted.

Package managers I used include:

pip (system wide and within conda; I have five pips ...)
apt (did not figure out what it did until too late)
conda
Where do these package managers install packages?

pip: https://stackoverflow.com/questions/29980798/where-does-pip-install-its-packages
When used without conda environment, will show the base python dir
pip when used with virtualenv will generally install packages in the path <virtualenv_name>/lib/<python_ver>/site-packages. For example, I created a test virtualenv named venv_test with Python 2.7, and the django folder is in venv_test/lib/python2.7/site-packages/django.
conda will install in the environment's python site-packages.
If there is no Python2 but we want a python2 package, conda seems will install it to the system-wide python site-packages
A detailed comparison about pip and apt:

pip is used to download and install packages directly from PyPI. PyPI is hosted by Python Software Foundation. It is a specialized package manager that only deals with python packages.

apt-get is used to download and install packages from Ubuntu repositories which are hosted by Canonical.

Some of the differences between installing python packages from apt-get and pip are as follows:

Canonical only provides packages for selected python modules. Whereas, PyPI hosts a much broader range of python modules. So, there are a lot of python modules which you won't be able to install using apt-get.
Canonical only hosts a single version of any package (generally the latest or the one released in recent past). So, with apt-get we cannot decide the version of python-package that we want. pip helps us in this situation. We can install any version of the package that has previously been uploaded on PyPI. This is extremely helpful in case of conflict in dependencies.
apt-get installs python modules in system-wide location. We cannot just install modules in our project virtualenv. pip solves this problem for us. If we are using pip after activating the virtualenv, it is intelligent enough to only install the modules in our project virtualenv. As mentioned in previous point, if there is a version of a particular python package already installed in system-wide location, and one of our project requires an older version of the same python package, in such situations we can use virtualenv and pip to install that older version of python package without any conflicts.
As @Radu Rădeanu pointed out in this answer, there would generally be difference in names of packages as well. Canonical usually names Python 2 packages as python-<package_name> and Python 3 packages as python3-<package_name>. Whereas for pip we generally just need to use <package_name> for both Python 2 as well as Python3 packages.
Which one should you use:
Both apt-get and pip are mature package managers which automatically install any other package dependency while installing. You may use anyone as you like. However, if you need to install a particular version of python-package, or install the package in a virtualenv, or install a package which is only hosted on PyPI; only pip would help you solve that issue. Otherwise, if you don't mind installing the packages in system-wide location it doesn't really matter whether you use apt-get or pip.

Also, this link that summarizes this problem perfectly: https://stackoverflow.com/questions/14117945/too-many-different-python-versions-on-my-system-and-causing-problem

 

The third issue is I confused Python packages with ROS packages somewhere in my projects. This is more conceptual but did misguide my actions. Here are some clarifications:

ROS

ROS is a programming ecosystem/framework that provide middleware for modules (packages) to communicate, especially for the purpose of robotics systems
ROS provide distribution in Python and C++
ROS code is organized by packages: folders containing src, data, other libraries...
ROS has a $ROS_PACKAGE_PATH, which is an environment variable indicating where to find packages for ROS. For example: `echo $ROS_PACKAGE_PATH` will give the output /opt/ros/kinetic/share https://answers.ros.org/question/241528/what-does-environment-set-up-do/
Python

Python path: environment variables. Where to find import packages...https://realpython.com/python-modules-packages/ the module search path
modules: individual scripts
package: a collection of modules https://realpython.com/python-modules-packages/#the-dir-function
Given this structure, if the pkg directory resides in a location where it can be found (in one of the directories contained in sys.path), you can refer to the two modules with dot notation (pkg.mod1, pkg.mod2) and import them with the syntax you are already familiar with
The __init__.py file in a package will import the modules into the package namespace, so that we can directly use imported modules in the main function.
 

Finally, some useful scripts for inspection and cleaning:

Find all Python:  sudo find / -type f -executable -iname 'python*' -exec file -i '{}' \; | awk -F: '/x-executable; charset=binary/ {print $1}' | xargs readlink -f | sort -u | xargs -I % sh -c 'echo -n "%: "; % -V'
`whereis pip`   finds all pips