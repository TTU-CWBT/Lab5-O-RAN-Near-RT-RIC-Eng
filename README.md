# Lab 5: Smart Base Station Control Platform and Mobility Management Experiment

## Table of Contents
- [1. Experiment Content](#1-experiment-content)
- [2. Related Knowledge](#2-related-knowledge)
- [3. Environment and Equipment Specifications](#3-environment-and-equipment-specifications)
- [4. Experiment Architecture](#4-experiment-architecture)
- [5. xApp Code Explanation](#5-xapp-code-explanation)
- [6. Experiment Steps](#6-experiment-steps)
- [7. References](#references)

## 1. Experiment Content
This experiment primarily focuses on learning how to set up and deploy the O-RAN RIC, as well as using an xApp and observing the results.

### 1-1 Questions and Discussion
1. Take a screenshot of the successful deployment screen from step 6-2 showing `watch kubectl get pod -A -o wide`.
2. Take a screenshot of the success screens for Terminal 1, 2, and 3 from the end of step 6-5.
3. Explain the physical meaning of the triggering and stopping conditions for the **A3 Event**.
4. Explain the physical meaning of the **Ping Pong Effect**.

## 2. Related Knowledge
- **O-RAN Near-RT RIC & E2AP:** The Near-RT RIC interacts with base stations (or Emulators) via the E2 Interface / E2AP. The process usually includes:
    - **E2 Setup:** The E2 Node establishes a connection with the RIC and exchanges capability information.
    - **Subscription:** The RIC subscribes to the measurements/events you need.
    - **Indication:** The E2 Node continuously reports measurement data (the xApp makes decisions based on this).
- **Deployment Related:**
    - **Kubernetes (K8s) / Helm:** RIC platform components and xApps run on K8s.
    - **AppMgr / DMS CLI:** Manages xApp onboarding, installation, and uninstallation.
    - **ChartMuseum:** The local repo for xApp Helm charts.
    - **Docker Registry:** Allows K8s to pull the xApp images you build.

## 3. Environment and Equipment Specifications
For this experiment, you can use either a physical host or a virtual machine. However, please note that Ubuntu must be version **20.04**. This is different from before, so please pay attention!

| Components  | Requirements               |
| ----------- | -------------------------- |
| Computer    | Physical / Virtual Machine |
| Ubuntu      | 20.04                      |
| Near RT-RIC | i-release                  |
| CPU         | 4 Core                     |
| RAM         | 8G                         |
| Storage     | 75 G                       |

## 4. Experiment Architecture
### 4-1 Experiment Environment Simulation Diagram
This experiment simulates the Handover xApp in the Near-RT RIC. After receiving real-time signals from the E2 Termination, it continuously makes decisions for the current environment and hands them over to the E2 Termination for judgment.

The field for judgment is shown in the image below. In an environment of 8000m x 6000m, there are 18 base stations, divided into an office area and a residential area. When a User Equipment (UE) moves between these two areas, the xApp determines if the UE needs to handover and provides the handover decision.
![screenshot_20260104_175135_(region)](https://hackmd.io/_uploads/BkAgK2PN-x.png)

### 4-2 Near-RT RIC Architecture
- **Emulator:** Responsible for simulating base station and UE related data.
- **Handover xApp:** Uses algorithms to analyze and decide whether to perform a handover.

![Screenshot 2025-12-29 at 12.25.29 PM](https://hackmd.io/_uploads/rkPpGt1V-x.png)
> The above data is referenced from O-RAN SC.
> [Official Specifications Website](https://specifications.o-ran.org/specifications)

### 4-3 Emulator
The Emulator simulates the commuting situation in a Science Park.

#### Environment Size
- Length: 8000m, Width: 6000m
- Number of Base Stations: 18
- Base Station Radius: 1000m

#### UE Settings
1. Movement is restricted within the rectangular area.
2. Simulation time: 24 hours.
3. Distribution scenario:

| **Shift Schedule** | **Work Time** | **Overall UE Percentage** |
| ------------------ | ------------- | --- |
| Night Shift        | 00:00 ~ 09:00 | 20% |
| Evening Shift      | 16:00 ~ 01:00 | 20% |
| Normal Day Shift   | 08:00 ~ 17:00 | 60% |

![screenshot_20260104_171726_(region)](https://hackmd.io/_uploads/Hkglq2DV-e.png)

## 5. xApp Code Explanation
![screenshot_20260104_171411_(region)](https://hackmd.io/_uploads/BJgF5nvVZx.png)
![screenshot_20260104_171335_(region)](https://hackmd.io/_uploads/BkJM5nDVWx.png)

## 6. Experiment Steps
### 6-0 Software Updater
After logging into Ubuntu, you need to update and install necessary tools. Complete the following commands in order:
```shell=1
sudo apt update
sudo apt upgrade
sudo apt install ubuntu-desktop
sudo -i
sudo apt install unrar
sudo apt-get install -y git vim curl net-tools openssh-server python3-pip nfs-common libsctp1
sudo apt-get update
```

### 6-1 Download O-RAN Near RT RIC Source Code
Unzip the compressed file and move it to the Desktop.
![image](https://hackmd.io/_uploads/B1-6Xjk4Wg.png)

### 6-2 Install O-RAN Near RT RIC

1. Install O-RAN SC I-Release
```shell=1
sudo -i
git clone https://gerrit.o-ran-sc.org/r/ric-plt/ric-dep -b i-release
ls ~/ric-dep/bin  # Check if ric-dep/bin is installed successfully
```
![image](https://hackmd.io/_uploads/HJGM4sk4Zx.png)

2. Overwrite the original files with `install_k8s_and_helm.sh`, `install_kong.sh`, and `install_RIC.sh` provided in the lesson materials (assuming you have placed them in the Desktop).
```shell=1
cd /home/*your_username*/Desktop 
cp install_RIC.sh ~/ric-dep/bin/
cp install_k8s_and_-_helm.sh ~/ric-dep/bin/
cp install_kong.sh ~/ric-dep/bin/
```

3. Grant permissions to these three Shell Scripts in order.
```shell=1
chmod 777 install_k8s_and_helm.sh 
chmod 777 install_kong.sh
chmod 777 install_RIC.sh
```
After successfully given permissions, the permission list will look like this:
![image](https://hackmd.io/_uploads/Hkh9-fl4be.png)

4. Execute the three shell scripts to install and deploy the necessary pods.

:warning: Please check if the IP in `./install_k8s_and_helm.sh` matches your current computer's IP first. If it is different, please modify it yourself! Otherwise, the installation will fail!
> You can check the local IP using the `ip a` command.
> ![image](https://hackmd.io/_uploads/SkVyxbe4-l.png)

:warning: Executing the following commands will take a lot of time, please be patient! If the virtual machine performance is poor, it will take even longer! It is normal to see repeated messages during installation, please do not worry.
```shell=1
./install_k8s_and_helm.sh 
./install_kong.sh
./install_RIC.sh
```
- Completion Screen:

![2025-12-27_09-09](https://hackmd.io/_uploads/Skp6EsJNWl.png)

After completing, check the deployment status of the pods using the following command:
```shell=1
watch kubectl get pod -A -o wide
```
![2025-12-27_09-09_1](https://hackmd.io/_uploads/SydlrskN-e.png)
If the result of `watch kubectl get pod -A -o wide` is normal, please skip step 5.

5. ~~Install O-RAN Near RT RIC Failed~~
If the installation fails during step 4, you will need to uninstall O-RAN Near RT RIC and redeploy it (hopefully, everyone won't need to reach this step).
- Uninstall O-RAN Near RT RIC command:
```shell=1
./uninstall
```
- Redeploy O-RAN Near RT RIC command:
```shell=1
cd ~/ric\-dep/bin
./install -f ../RECIPE_EXAMPLE/example_recipe_oran_i_release.yaml -c "jaegeradapter influxdb"
```

### 6-3 xApp Deployment Tool (DMS CLI) Installation
O-RAN SC provides a deployment tool called "**DMS CLI**". You only need to use the following commands to successfully deploy the xApp.
```shell=1
docker run --rm -u 0 -it -d -p 8090:8080 -e DEBUG=1 -e STORAGE=local -e STORAGE_LOCAL_ROOTDIR=/charts -v $(pwd)/charts:/charts chartmuseum/chartmuseum:latest
export CHART_REPO_URL=http://0.0.0.0:8090

git clone https://gerrit.o-ran-sc.org/r/ric-plt/appmgr -b h-release
cd appmgr/xapp_orchestrater/dev/xapp_onboarder

apt-get install python3-pip
pip3 uninstall xapp_onboarder
pip3 install ./

chmod 755 /usr/local/bin/dms_cli
ls -la /usr/local/lib/python3.8

chmod -R 755 /usr/local/lib/python3.8
```
After entering the commands, you need to check if the entire xApp deployment toolchain (DMS/xapp_onboarder) works properly. If it does, it will display `True`.
```shell=1
dms_cli health
```
![image](https://hackmd.io/_uploads/BJotrsyV-l.png)

### 6-4 xApp File Modification
Because different pods are assigned different IPs when deploying the RIC, you need to find the Dockerfile in `ric-app-A3handover` and modify the `ENV SUBMGR` IP to your own. You can find your `submgr` IP using the previous `watch kubectl get pod -A -o wide` command.
>![image](https://hackmd.io/_uploads/B1pWZMlEWg.png)
>![image](https://hackmd.io/_uploads/HJL8bzlEWl.png)

### 6-5 Connecting RIC, Emulator, and xApp
0. In this step, you will need to open three terminals first, and enter the path using the following commands (Remember! This is required for all three terminals).
```shell=1
sudo -i
cd /home/<your username>/Desktop
```
1. Execute the following commands in the third terminal first.

:warning: Please pay attention to the last curl command. In `http://<appmgr IP>:8080/ric/v1/register`, you need to use `watch kubectl get pod -A -o wide` to find the IP of appmgr.
For example: `http://10.244.0.11:8080/ric/v1/register`
```shell=1
cd ric-app-A3handover
docker run -d -p 5000:5000 --restart=always --name registry registry:2
export CHART_REPO_URL=http://0.0.0.0:8090
dms_cli onboard ./init/config-file.json ./init/schema.json
docker build -t 127.0.0.1:5000/hw-python:1.0.2 .
docker tag 127.0.0.1:5000/hw-python:1.0.2 127.0.0.1:5000/hw-python:1.0.2
docker push 127.0.0.1:5000/hw-python:1.0.2

curl -v -X POST 'http://<appmgr IP>:8080/ric/v1/register' -H 'accept:application/json' -H 'Content-Type:application/json' -d '@hwpython-register.json'
```
> Success Screen
> ![2025-12-27_09-29](https://hackmd.io/_uploads/S1x4SiJNZe.png)
2. Grant permissions to the Shell Script files in order.
```shell=1
chmod 777 Emulator_TCP
chmod 777 emulator_connect_run.sh
chmod 777 emulator_run.sh
```
3. Establish the settings required for Kubernetes.
```shell=1
mkdir -p ~/.kube
sudo cp -i /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```
4. Execute the following commands in the three terminals **in order**.
```shell=1
# Execute in the first terminal. Wait for "wait for connect..." to appear before running the second command.
./emulator_run.sh 

# Execute in the second terminal 
./emulator_connect_run.sh

# Execute in the third terminal 
dms_cli install hw-python 1.0.1 ricxapp
```
If you see the following screens, the connection is successful!

:warning: Be sure to take screenshots when successful!
- **Terminal 1** will display the current situation and whether a handover is needed.

![image](https://hackmd.io/_uploads/SJ1LA91V-g.png)
- **Terminal 2**: Check the red box (E2 Setup status) to see if the Emulator is working properly, and the yellow box (RIC Subscription status) to see if the xApp is working properly.

![success](https://hackmd.io/_uploads/r18m1jyEZg.png)
- **Terminal 3** will display OK.

![image](https://hackmd.io/_uploads/rk25ks14Zl.png)

5. Clear ricxapp
- If you find that you cannot connect or the above two procedures are not completed successfully, please interrupt Terminal 1 and Terminal 2, and use the following command to uninstall the xApp deployment.
```shell=1
dms_cli uninstall hw-python ricxapp
```

## References
- [Ministry of Education 5G Technology Alliance Module 13](https://drive.google.com/drive/folders/1xWStVcOl4KDpCvO0nRylCXAnWFBBBY3g)
