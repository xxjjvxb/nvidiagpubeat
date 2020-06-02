개인의 사용의 편의를 위해 포크를 떠서 편집.
메모의 기능.

1. 깃 클론에서의 링크 수정.
 - 수정된 README.md에 반영 되었음.
2. service가 수행할 script 작성 # service_nvidiagpubeat.sh
```bash                                                              
echo -e `date` " : Start nvidia-gpu Systemd Test " >> ~/daemon_monitor.txt           
export PATH=$PATH:.

while :
do
    /home/lee19/utils/beats_dev/src/github.com/ebay/nvidiagpubeat   = c nvidiagpubeat.yml -e -d "*" =E seccomp.enabled=false
done     
```
3. systemd 에 Service 등록
systemd 에 서비스를 등록하기 위해 아래 경로에 아래와 같이 설정을 합니다.
```
vi /etc/systemd/system/nvbeat.service

# /etc/systemd/system/nvbeat.service 내용
[Unit]
Description=Systemd NVIDIA-GPU-BEAT Daemon
[Service]
Type=simple
ExecStart=/path/to/nvidiagpubeat/test-daemon.sh
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

서비스 등록 및 시작
+ 등록된 서비스 시작
```# systemctl start nvbeat```

// 서비스 상태 체크
```# systemctl status nvbeat```
+ 재부팅 후에도 서비스가 시작되도록 서비스 등록
```# systemctl enable testchk```

https://chhanz.github.io/linux/2019/01/18/linux-how-to-create-custom-systemd-service/ 

!--
This module was automatically generated using the framework found below:
https://www.elastic.co/guide/en/beats/devguide/current/new-beat.html

Modifications to auto-generated code - Copyright 2018 eBay Inc.
Architect/Developer: Deepak Vasthimal

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# nvidiagpubeat

Welcome to nvidiagpubeat.
nvidiagpubeat is an elastic beat that uses NVIDIA System Management Interface (nvidia-smi - https://developer.nvidia.com/nvidia-system-management-interface) to monitor NVIDIA GPU devices and can ingest metrics into Elastic search cluster. nvidia-smi is a command line utility, based on top of the NVIDIA Management Library (NVML), intended to aid in the management and monitoring of NVIDIA GPU devices.

nvidiagpubeat is built using Beats framework described at https://www.elastic.co/guide/en/beats/devguide/current/new-beat.html.

nvidiagpubeat elastic beat with help of nvidia-smi allows administrators to query GPU device state.  It is targeted at the TeslaTM, GRIDTM, QuadroTM and Titan X product, though limited support is also available on other NVIDIA GPUs.

NVIDIA-smi ships with NVIDIA GPU display drivers on Linux, and with 64bit Windows Server 2008 R2 and Windows 7.

nvidiagpubeat provides ability (look at nvidiagpubeat.yml) to configure metrics that needs to be monitored and by default it queries utilization.gpu,utilization.memory,memory.total,memory.free,memory.used,temperature.gpu,pstate and can ingest them into elastic search cluster for possibly visualization using Kibana.

## Getting Started with nvidiagpubeat

### Build on macOS
#### Prerequisites

* [Golang](https://golang.org/dl/) 1.7+
* virtualenv - https://virtualenv.pypa.io/en/stable/
```
sudo curl https://bootstrap.pypa.io/get-pip.py | sudo python
sudo pip install virtualenv
```
* glide - https://github.com/Masterminds/glide#install
```
brew install glide
```

### Build on RedHat EL7
#### Prerequisites
```
yum install python-virtualenv
yum install golang
```

### Initialize Project
Once the prerequisites have been installed, rest of the steps are common across OSes
as `nvidiagpubeat` is written in Golang.

```
#Start with an empty directory
mkdir beats_dev

#Build with elastic beats branch=6.5 (Master branch might not always upto-date).
export WORKSPACE=`pwd`/beats_dev
export GOPATH=$WORKSPACE
git clone https://github.com/elastic/beats ${GOPATH}/src/github.com/elastic/beats --branch 6.5

#Clone nvidiagpubeat
mkdir $WORKSPACE/src/github.com/ebay
cd $WORKSPACE/src/github.com/ebay
git clone https://github.com/xxjjvxb/nvidiagpubeat.git

#Build
cd $WORKSPACE/src/github.com/ebay/nvidiagpubeat/
make setup
make
```
### nvidiagpubeat and beats compatibility
To keep `nvidiagpubeat` compatible with upcoming versions of `beats`, `nvidiagpubeat` will be branched out beacause
at times `beats` does make breaking changes across major releases. For instance between `beats 6.x` and `beats 7.x`, the
`cmd.GenRootCmd` is broken.

| beats version| nvidiagpubeat branch|
|---|---|
| 6.x | master |
| 7.x | withBeats7.3 |

For instance to use 7.3 version of `beats` and compatible `nvidiagpubeat`, perform following actions before compilation.

```
#Build with elastic beats branch=6.5 (Master branch might not always upto-date).
export WORKSPACE=`pwd`/beats_dev
export GOPATH=$WORKSPACE
git clone https://github.com/elastic/beats ${GOPATH}/src/github.com/elastic/beats --branch 7.3

#Clone nvidiagpubeat
mkdir $WORKSPACE/src/github.com/ebay
cd $WORKSPACE/src/github.com/ebay
git clone https://github.com/xxjjvxb/nvidiagpubeat.git --branch withBeats7.3
```


Above instructions will generate a binary in the same directory with the name `nvidiagpubeat`.

### Run in production environment

To run nvidiagpubeat with pre-installed nvidia-smi that is available in PATH, switch to "test" environment
in nvidiagpubeat.yml and use

```yaml
env: "production"
```

```bash
export PATH=$PATH:.
./nvidiagpubeat -c nvidiagpubeat.yml -e -d "*" -E seccomp.enabled=false
```

`seccomp.enabled` setting : nvidiagpubeat uses libbeat framework. For security purposes the libbeat framework by default
drops the ability to fork/exec. As nvidiagpubeat executes `nvidia-smi`, security setting must be disabled by
setting through command line.

### Run in test environment (macOS)
To run nvidiagpubeat with pre-packaged `localnvidiasmi` switch to "test" environment in nvidiagpubeat.yml and use
```yaml
env: "test"
```

```bash
export PATH=$PATH:.
./nvidiagpubeat -c nvidiagpubeat.yml -e -d "*" -E seccomp.enabled=false
```

localnvidiasmi executable built for macOS and is a mock GPU event generator.


### Sample event
Below is a sample event emitted by nvidiagpubeat.

```
Publish event: {
  "@timestamp": "2019-03-25T15:34:17.739Z",
  "@metadata": {
    "beat": "nvidiagpubeat",
    "type": "doc",
    "version": "6.5.5"
  },
  "utilization": {
    "gpu": 4,
    "memory": 40
  },
  "memory": {
    "used": 0,
    "total": 6082,
    "free": 6082
  },
  "temperature": {
    "gpu": 27
  },
  "pstate": 8,
  "beat": {
    "name": "hostname.company.com",
    "hostname": "hostname.company.com",
    "version": "6.5.5"
  },
  "host": {
    "name": "hostname.company.com"
  },
  "gpuIndex": 3,
  "type": "nvidiagpubeat"
}
```

### Build

Make changes (if any) and build the binary for nvidiagpubeat run the command below. This will generate a binary
in the same directory with the name `nvidiagpubeat`.

```
make
```

### Test cases

To test nvidiagpubeat, run the following command:

```
make unit-tests
make integration-tests
make coverage-report
```

The test coverage is reported in the folder `./build/coverage/`

### Cleanup dependencies
To remove them dependent libraries, builds, temporary files, files created during tests and the executatble, simply run

```
 rm -rf build/
 rm -rf data/
 rm -rf nvidiagpubeat
 rm -rf nvidiagpubeat.test
 rm -rf vendor
 rm -rf logs
```

### Clone

To clone nvidiagpubeat from the git repository, run the following commands:

```
mkdir -p ${GOPATH}/github.com/ebay
cd ${GOPATH}/github.com/ebay
git clone https://github.com/xxjjvxb/nvidiagpubeat.git
cd nvidiagpubeat
git remote add upstream https://github.com/xxjjvxb/nvidiagpubeat.git
```

For further development, check out the [beat developer guide](https://www.elastic.co/guide/en/beats/libbeat/current/new-beat.html).

### Testimonials
I would love to hear about your use case. It will help me improve `nvidiagpubeat`. Please add few lines about
your use case, affiliation and location. All fields are optional.

1. Use Case: I am running deeplearning models on a headless linux machine. Therefore, it is essential for me to track my gpu load and related stats.

   Country: Germany

   Full Name: Julius Zimmermann

   Affiliation: Student

2. Use Case:  

   Country:

   Full Name:

   Affiliation:

3. Use Case:  

   Country:

   Full Name:

   Affiliation:


### License
Copyright 2016-2018 eBay Inc.
Architect/Developer: Deepak Vasthimal

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
