FROM centos:7
MAINTAINER spli@dundee.ac.uk

RUN yum install -y epel-release
RUN yum install -y python-pip
RUN pip install omego

# This is the proper way to install Ice Python:
#RUN pip install zeroc-ice
# But this is a shortcut that installs a precompiled version
RUN pip install https://github.com/openmicroscopy/zeroc-ice-py-centos7/releases/download/0.0.3/zeroc_ice-3.6.4-cp27-cp27mu-linux_x86_64.whl
RUN pip install pillow

RUN useradd omero
WORKDIR /home/omero
USER omero
RUN omego download python --ice 3.6 --sym OMERO.py

CMD ["/opt/omero/OMERO.py/bin/omero", "web", "start", "--foreground"]
