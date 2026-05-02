# Qwen3 on Intel NPU - Fedora 44

This setup was tested on Fedora 44 KDE, installed on laptop with Intel® Core™ Ultra 7 Processor 258V (code name Lunar Lake), but should also work on any Lunar Lake processor, due to low memory demands of Qwen3 model. Lunar Lake is very efficient and fast platform (it uses high bandwith unified memory), so it is an excelent platform for running local AI models.

Hardware: Laptop with Intel® Core™ Ultra 7 Processor 258V (code name Lunar Lake)
<br>
<br>
Operating System: Fedora KDE Plasma Desktop 44 kernel version: 6.19.14-300.fc44.x86_64
<br>
<br>
Driver version: Intel - linux-npu-driver v1.32.1



[Qwen3 in Docker setup](#qwen3-in-docker-setup---for-intel-lunar-lake-npu)

[Connecting VS Code extension "Continue" to local Qwen model](#connecting-visual-studio-code-extension-continue-to-local-qwen-model)




<br>
<br>

<h1>Qwen3 in Docker on intel NPU as a local model for Visual Studio Code - installation on Fedora 44</h1>


## Qwen3 in Docker setup - For Intel Lunar Lake NPU

After installation of Fedora 44 open terminal and do update command:

```bash
sudo dnf update
```

To be able to pull and compile source code for Intel’s npu-linux-driver that has no binary relase, you should install “git”.

```bash
sudo dnf install git-all
```

I will do my work in /home/user directory, so if you don’t want to do process with your current user, to be able to just copy/paste the code, just navigate to your user folder, by running command:

```bash
cd
```

After successful installation of “git”, pull the latest working source code. Do that by checking the github repo where you can find the drop-down list in the left corner and then choose appropriate version. In Branches tab, that version is “dev/jwludzik/snapshot-v1.32.1”, and in the Tags tabs that version is v1.32.1, but they are basically pointing to the same source code. In my case, I will pull the source code from latest working branch.

```bash
git clone -b dev/jwludzik/snapshot-v1.32.1 --recurse-submodules --depth 1 https://github.com/intel/linux-npu-driver.git
```

After you clone source code successfully, change your current folder to linux-npu-driver by running command:

```bash
cd linux-npu-driver
```

Because the project configuration is done for cmake, you should download cmake by running:

```bash
sudo dnf install cmake
```

Also download compiling tools for Fedora, by running command:

```bash
sudo dnf install @development-tools @c-development
```

Now build the packages by running cmake command bellow
I used:  -DCMAKE_POLICY_VERSION_MINIMUM=3.5 to avoid compatibility issues.

```bash
cmake -B build -DCMAKE_POLICY_VERSION_MINIMUM=3.5
```

If you are working this on unplugged laptop (which is not unusual for NPU scenario), set your setting for sleep on battery to more time – depending on your internet speed, so that machine is able to run “make” command without sleep interruption.

```bash
make
```

After make command successfully builds binaries and libraries, install them into users bin and lib directories by running command:

```bash
cmake --install . --prefix ~/.local
```

Now when you installed binary and library files into user directories, put them on the path by running command for binaries first:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

And then for librar too:

```bash
export LD_LIBRARY_PATH="$HOME/.local/lib64:$LD_LIBRARY_PATH"
```

Reload the intel_vpu module to load new firmware:

```bash
sudo rmmod intel_vpu
sudo modprobe intel_vpu
```


Now you should be able to run test:

```bash
npu-kmd-test
```

Create directory for models:

```bash
mkdir -p $(pwd)/models
```
Change permissions for previously created models folder to 700

```bash
sudo chmod -R 700 $(pwd)/models
```
<br>
<br>
<br>

**Install DOCKER for fedora (follow tutorial from their web, avoid dnf install)**

https://docs.docker.com/engine/install/fedora/

Add repository:

```bash
sudo dnf config-manager addrepo --from-repofile https://download.docker.com/linux/fedora/docker-ce.repo
```

Install the packages:

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Enable docker:

```bash
sudo systemctl enable --now docker
```

Pull the OpenVINO mode server docker image (I choosed latest-py version because it was recently relased and supports NPU-s):
```bash
sudo docker pull openvino/model_server:latest-py
```

After you pulled model, run docker model with command bellow, to also download the Qwen3 model, and run everything.

**Running docker container without detached flag so you can see logs (Version 1)**

For first run, I suggest running container without detached (-d) option, so you can see logs if something goes wrong, because it will take time to download and setup the model:

```bash
sudo docker run --user $(id -u):$(id -g) --device /dev/dri --group-add=$(stat -c "%g" /dev/dri/render* | head -n 1) --device /dev/accel/accel0:/dev/accel/accel0 --device /dev/dri:/dev/dri --rm -p 8000:8000 -v $(pwd)/models:/models:rw openvino/model_server:latest-py --source_model OpenVINO/Qwen3-8B-int4-cw-ov --model_repository_path models --task text_generation --rest_port 8000 --target_device NPU
```

**Or just run it with detached option (-d) and look for logs afterwards (Version 2):**

```bash
sudo docker run --user $(id -u):$(id -g) -d --device /dev/dri --group-add=$(stat -c "%g" /dev/dri/render* | head -n 1) --device /dev/accel/accel0:/dev/accel/accel0 --device /dev/dri:/dev/dri --rm -p 8000:8000 -v $(pwd)/models:/models:rw openvino/model_server:latest-py --source_model OpenVINO/Qwen3-8B-int4-cw-ov --model_repository_path models --task text_generation --rest_port 8000 --target_device NPU
```


Check if container is running, with command:

```
sudo docker ps
```

and after you get CONTAINER ID, just run:

```
sudo docker logs -f container_id_here
```

<br>
<br>

## Connecting Visual Studio Code extension "Continue" to local Qwen model

**Install VISUAL STUDIO CODE for Fedora (follow tutorial from their web)**

Add Microsoft repository:
```
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc &&
echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\nautorefresh=1\ntype=rpm-md\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" | sudo tee /etc/yum.repos.d/vscode.repo > /dev/null
```

Install Visual Studio Code:
</h2>
dnf check-update &&
sudo dnf install code # or code-insiders


After installation Start Visual Studio Code:

```CTRL + SHIFT + x``` to extend extension sidebar, and search for extension named “Continue” and download the extension.

Open Continue extension and in the top right corner of extension you should see a drop down menu with config selection options. There choose settings wheel right to the **Local Config**.


If you only plan to use only Qwen model, just copy and replace the whole config:


```yaml
name: Local Config
version: 1.0.0
schema: v1
models:
  - name: "Qwen NPU"
    provider: "openai"
    model: "OpenVINO/Qwen3-8B-int4-cw-ov"
    apiBase: "http://localhost:8000/v3"
    apiKey: "unused"
    completionOptions:
    stream: false
    roles:
      - chat
      - edit
      - apply
      - autocomplete
```

And if you plan to use other models too, copy only code that is bellow **models:** in above snippet.
