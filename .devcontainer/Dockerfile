FROM effibot/cuda-ubuntu16:latest
# default user is dock (non-root but with sudo privileges) with UID and GID 1000
# and home directory /home/dock

# Create a python2.7 virtual environment for NAOqi
RUN mamba create -n nao python=2.7 -y && \
    mamba init zsh && mamba init bash

RUN mamba install -n nao -y \
    autopep8 bandit matplotlib\
    numpy opencv scipy pylint && \
    mamba update --all -y && mamba clean --all --yes && \
    echo "mamba activate nao" >> ~/.zshrc && \
    echo "mamba activate nao" >> ~/.bashrc && \
    echo "alias nao='mamba activate nao'" >> ~/.aliases.sh && \
    echo "alias base='mamba activate base'" >> ~/.aliases.sh

# install packages for ROS-Kinetic
RUN sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu xenial main" > /etc/apt/sources.list.d/ros-latest.list' && \
    curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add - && \
    sudo apt-get update && sudo apt install -y ros-kinetic-desktop-full build-essential \
    ros-kinetic-move-base-msgs ros-kinetic-octomap ros-kinetic-octomap-msgs \
    ros-kinetic-humanoid-msgs ros-kinetic-humanoid-nav-msgs \
    ros-kinetic-camera-info-manager ros-kinetic-camera-info-manager-py \
    ros-kinetic-rosbridge-suite ros-kinetic-rosbridge-server ros-kinetic-rosbridge-library \
    ros-kinetic-rosbridge-msgs

RUN sudo apt autoremove -y && sudo apt clean -y && \
    sudo rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*/apt/lists/* && \
    echo "source /opt/ros/kinetic/setup.zsh" >> ~/.zshrc

# add additional ROS-python packages then initialize rosdep
RUN mamba install -n nao -y defusedxml rosdep rosinstall rosinstall_generator wstool catkin_tools && \
    mamba update -n nao --all -y && mamba clean --all --yes && \
    sudo rosdep init && rosdep update --include-eol-distros

# Add needed folder to environment variables
ENV NAOQI=${HOME}/naoqi
ENV PYTHONPATH=${NAOQI}/pynaoqi/lib/python2.7/site-packages:${PYTHONPATH}
ENV AL_DIR=${NAOQI}/naoqi-sdk
ENV AL_DIR_SIM=${NAOQI}/naoqi-sdk

# download and install NAOqi SDK
RUN mkdir ${HOME}/naoqi && \
    wget https://community-static.aldebaran.com/resources/2.5.10/Python%20SDK/pynaoqi-python2.7-2.5.7.1-linux64.tar.gz -P /tmp && \
    wget https://community-static.aldebaran.com/resources/2.5.10/NAOqi%20SDK/naoqi-sdk-2.5.7.1-linux64.tar.gz -P /tmp && \
    tar -xvzf /tmp/pynaoqi-python2.7-* -C ${NAOQI} && mv ${NAOQI}/pynaoqi-* ${NAOQI}/pynaoqi && \
    tar -xvzf /tmp/naoqi-sdk-* -C ${NAOQI} && mv ${NAOQI}/naoqi-sdk-* ${NAOQI}/naoqi-sdk && \
    rm -rf /tmp/pynaoqi-python2.7-* && \
    rm -rf /tmp/naoqi-sdk-*

# install ROS-Pepper interface
RUN echo 'debconf debconf/frontend select Noninteractive' | sudo debconf-set-selections &&\
    echo 'ros-pepper-meshes ros-pepper-meshes/accepted-ros-pepper-meshes boolean true' | sudo debconf-set-selections &&\
    sudo apt-get update && sudo apt install -y ros-kinetic-pepper-bringup ros-kinetic-pepper-dcm-bringup \
    ros-kinetic-pepper-description ros-kinetic-pepper-gazebo-plugin \
    ros-kinetic-pepper-moveit-config ros-kinetic-pepper-robot \
    ros-kinetic-pepper-meshes && \
    sudo apt autoremove -y && sudo apt clean -y && \
    sudo rm -rf /var/lib/apt/lists/* /tmp/*

# Install NAOqi ROS dependencies
ENV ROS_DISTRO=kinetic
ENV ROS_PYTHON_VERSION=2.7

RUN mkdir -p ~/catkin_ws/src && cd ~/catkin_ws/src && \
    git clone https://github.com/ros-naoqi/naoqi_driver.git && \
    cd ~/catkin_ws/src && rosdep install -i -y --from-paths ./naoqi_driver && \
    . /opt/ros/kinetic/setup.sh && \
    cd ~/catkin_ws && catkin_make