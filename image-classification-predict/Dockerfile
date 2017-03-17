FROM ubuntu:14.04

# Install build-essential, git, wget and other dependencies
RUN apt-get update && apt-get install -y \
  build-essential \
  git \
  libopenblas-dev \
  libopencv-dev \
  python-dev \
  python-numpy \
  python-setuptools \
  wget \
  python3-pip \
  nginx supervisor \
  python-pip \
  python-opencv


# Clone MXNet repo and move into it
RUN cd / && git clone --recursive https://github.com/dmlc/mxnet && cd mxnet && \
# Copy config.mk
  cp make/config.mk config.mk && \
# Set OpenBLAS
  sed -i 's/USE_BLAS = atlas/USE_BLAS = openblas/g' config.mk && \
# Make
  make -j"$(nproc)"

# Install Python package
RUN cd /mxnet/python && python setup.py install

# Add R to apt sources
RUN echo "deb http://cran.rstudio.com/bin/linux/ubuntu trusty/" >> /etc/apt/sources.list
# Install latest version of R
RUN apt-get update && apt-get install -y --force-yes r-base

#install Flask
RUN pip3 install uwsgi Flask
RUN pip install numpy


ADD ./app /app
ADD ./config /config

RUN pip install -r /config/requirements.txt


#RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
#RUN rm /etc/nginx/sites-enabled/default

#RUN ln -s /config/nginx.conf /etc/nginx/sites-enabled/
#RUN ln -s /config/supervisor.conf /etc/supervisor/conf.d/

EXPOSE 5000

#CMD ["supervisord", "-n"]
CMD ["python", "app/app2.py"]
