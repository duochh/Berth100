# Pixel Streaming Demo in Windows Docker container

- [Pixel Streaming Overview](https://docs.unrealengine.com/5.3/en-US/overview-of-pixel-streaming-in-unreal-engine/).
- [Pixel Streaming on running container](https://unrealcontainers.com/docs/use-cases/pixel-streaming).

## Prerequesites

- Container host system much follow requirements of [GPU acceleration in Windows containers](https://learn.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/gpu-acceleration#requirements).
- GPU on container host much follow requirements of [Pixel Streaming Reference](https://docs.unrealengine.com/5.3/en-US/unreal-engine-pixel-streaming-reference/).
- Container host much be installed [Docker Desktop on Windows](https://docs.docker.com/desktop/install/windows-install/).
- (Optional) Install [Windows Terminal](https://github.com/microsoft/terminal) on container host.

## Description

- Dockerfile.mcr:
  - Use Microsoft Official [Windows Server Image ltsc2022](https://hub.docker.com/_/microsoft-windows-server/).
  - Install the DirectX runtime files, the DirectX shader compiler files and the Visual C++ runtime files from [Offscreen rendering in Windows containers](https://unrealcontainers.com/blog/offscreen-rendering-in-windows-containers/#rendering-with-gpu-acceleration).
  - Enabling NVIDIA graphics APIs from [Enabling vendor-specific graphics APIs in Windows containers](https://unrealcontainers.com/blog/enabling-vendor-specific-graphics-apis-in-windows-containers/).
  - Copy project built package to container.
  - Run with GPU acceleration. Successfully!!!

## How to run Dockerfile.mcr

- Copy dockerfile and helper scripts to ue project folder.
- Update project path in docker file:

    ```dockerfile
    # Copy our Unreal Engine application into the container image here
    # (This assumes we have a packaged project called "MyProject" with a `-Cmd.exe` suffixed
    #  version as per the blog post "Offscreen rendering in Windows containers")
    RUN mkdir C:\projects\<MyProject>\Packaged\
    COPY \<MyProject>\ C:\projects\<MyProject>\Packaged\
    ```

- Build docker image from docker file.

    ```cmd
    docker build . -f <Dockerfile.*> -t <image-name>
    ```

- Run container from docker image and exec into it in powershell terminal.

    ```cmd
    docker run -it --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 --name <container-name> <image-name>
    docker ps -aqf "name=<container-name>"
    docker exec -it <container-id> powershell
    ```

- Run Pixel Streaming Infrastructrue Signalling Server.

    ```powershell
    .\PixelStreamingInfrastructure\SignallingWebServer\platform_scripts\cmd\Start_SignallingServer.ps1
    ```

- Run Unreal Engine Pixel Streaming package.

    ```powershell
    .\projects\Berth100\Packaged\Windows\Berth100\Binaries\Win64\Berth100.exe -PixelStreamingIP=127.0.0.1 -PixelStreamingPort=8888 -PixelStreamingUrl=ws://localhost:8888 -AllowPixelStreamingCommands -RenderOffScreen -StdOut -FullStdOutLogOutput
    ```

- Other way to run

    ```cmd
    docker push duochh/server-pixel-streaming-berth100:ltsc2022
    docker run -it --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 --name berth100 duochh/server-pixel-streaming-berth100:ltsc2022
    ```

- Verify running game in container:

    ```cmd
    docker exec -it <container-id> powershell
    ```

    ```powershell
    # Check running process and copy Id of game process
    Get-Process -ProcessName <MyProject>

    # Check using port of game process
    Get-NetTcpConnection -OwningProcess <MyProjectId>
    ```

## How to view on Browser

- Find docker container id:

    ```powershell
    docker ps -a
    ```

- Find docker container IP address:

    ```powershell
    docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container-id>
    ```

- Open browser and go to this link: <http://containerIpAddress/Showcase.html>
