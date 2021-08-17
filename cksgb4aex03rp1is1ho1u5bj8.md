## Multi-arch docker with buildx

## Introduction

Docker images can support multiple architectures, which means that a single image may contain variants for different architectures, and sometimes for different operating systems, such as Windows.

When running an image with multi-architecture support, docker automatically selects the image variant that matches your OS and architecture.

Most of the official images on Docker Hub provide a variety of architectures. For example, the busybox image supports `amd64`, `arm32v5`, `arm32v6`, `arm32v7`, `arm64v8`, `i386`, `ppc64le`, and `s390x`. When running this image on an `x86_64` / `amd64` machine, the `x86_64` variant is pulled and run.

## Tutorial

```bash
$ docker buildx ls                                                                                                                                                                                            

NAME/NODE     DRIVER/ENDPOINT STATUS                 PLATFORMS
desktop-linux                 protocol not available
default *     docker
  default     default         running                linux/amd64, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
```

```bash
$ docker buildx version                                                                                                                                                                                       

github.com/docker/buildx v0.5.1-docker 11057da37336192bfc57d81e02359ba7ba848e4a
```


```bash
$ docker buildx create --use --name mybuilder                                                                                                   

mybuilder
$ docker buildx ls                                                                                                                              

NAME/NODE     DRIVER/ENDPOINT             STATUS                 PLATFORMS
mybuilder *   docker-container
  mybuilder0  unix:///var/run/docker.sock inactive
desktop-linux                             protocol not available
default       docker
  default     default                     running                linux/amd64, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
```

main.go
```go
package main

import (
        "fmt"
        "runtime"
)

func main() {
        fmt.Printf("Hello, %s!\n", runtime.GOARCH)
}
```

Dockerfile
```Dockerfile
FROM golang:alpine AS builder
RUN mkdir /app
ADD . /app/
WORKDIR /app
RUN go mod init example.com/m
RUN go build -o hello .

FROM alpine
RUN mkdir /app
WORKDIR /app
COPY --from=builder /app/hello .
CMD ["./hello"]
```

```bash
$ docker buildx build -t hello-go-arch --platform=linux/arm,linux/arm64,linux/amd64 .                                                           

WARN[0000] No output specified for docker-container driver. Build result will only remain in the build cache. To push result image into registry use --push or to load image into docker use --load
[+] Building 56.1s (40/40) FINISHED
 => [internal] booting buildkit                                                                                                                5.3s
 => => pulling image moby/buildkit:buildx-stable-1                                                                                             4.3s
 => => creating container buildx_buildkit_mybuilder0                                                                                           1.0s
 => [internal] load build definition from Dockerfile                                                                                           0.1s
 => => transferring dockerfile: 253B                                                                                                           0.0s
 => [internal] load .dockerignore                                                                                                              0.1s
 => => transferring context: 2B                                                                                                                0.0s
 => [linux/amd64 internal] load metadata for docker.io/library/alpine:latest                                                                  12.0s
 => [linux/amd64 internal] load metadata for docker.io/library/golang:alpine                                                                  10.9s
 => [linux/arm64 internal] load metadata for docker.io/library/alpine:latest                                                                  11.6s
 => [linux/arm64 internal] load metadata for docker.io/library/golang:alpine                                                                  10.8s
 => [linux/arm/v7 internal] load metadata for docker.io/library/alpine:latest                                                                 11.6s
 => [linux/arm/v7 internal] load metadata for docker.io/library/golang:alpine                                                                 11.5s
 => [linux/arm/v7 builder 1/6] FROM docker.io/library/golang:alpine@sha256:8b27003087e1612c4a0ba06be640b2cfcebb5590c2de9869d88669f1a9da8286   31.5s
 => => resolve docker.io/library/golang:alpine@sha256:8b27003087e1612c4a0ba06be640b2cfcebb5590c2de9869d88669f1a9da8286                         0.1s
 => => sha256:e54b3edeab2b8da45b3e3b749ee47b38c7d2d96bf0fdc1abd0f59d42c9fd4acf 156B / 156B                                                     0.3s
 => => sha256:013899c35642552607b44e264c195a8a5cccff255eec04f422cd225eb3694d95 104.75MB / 104.75MB                                            26.0s
 => => sha256:edba13042e2b42dba9e7f0e57a60855dff15f2890b8d942f61fbdc4852a43854 154B / 154B                                                     0.3s
 => => sha256:adc7b26480300ed9c6eddb3c4eb897051a740425806a7edff4973e09fae2bb8f 280.81kB / 280.81kB                                             0.3s
 => => extracting sha256:adc7b26480300ed9c6eddb3c4eb897051a740425806a7edff4973e09fae2bb8f                                                      0.1s
 => => extracting sha256:edba13042e2b42dba9e7f0e57a60855dff15f2890b8d942f61fbdc4852a43854                                                      0.1s
 => => extracting sha256:013899c35642552607b44e264c195a8a5cccff255eec04f422cd225eb3694d95                                                      5.0s
 => => extracting sha256:e54b3edeab2b8da45b3e3b749ee47b38c7d2d96bf0fdc1abd0f59d42c9fd4acf                                                      0.2s
 => [internal] load build context                                                                                                              0.5s
 => => transferring context: 415B                                                                                                              0.0s
 => [linux/arm/v7 stage-1 1/4] FROM docker.io/library/alpine@sha256:eb3e4e175ba6d212ba1d6e04fc0782916c08e1c9d7b45892e9796141b1d379ae           1.0s
 => => resolve docker.io/library/alpine@sha256:eb3e4e175ba6d212ba1d6e04fc0782916c08e1c9d7b45892e9796141b1d379ae                                0.1s
 => => sha256:4ee0caa23b369b04827640a4be298bf4ff7bacd030c77e915f5d7fb8f987594a 2.43MB / 2.43MB                                                 0.8s
 => => extracting sha256:4ee0caa23b369b04827640a4be298bf4ff7bacd030c77e915f5d7fb8f987594a                                                      0.2s
 => [linux/amd64 builder 1/6] FROM docker.io/library/golang:alpine@sha256:8b27003087e1612c4a0ba06be640b2cfcebb5590c2de9869d88669f1a9da8286    32.0s
 => => resolve docker.io/library/golang:alpine@sha256:8b27003087e1612c4a0ba06be640b2cfcebb5590c2de9869d88669f1a9da8286                         0.2s
 => => sha256:4ac7b6f906a9a07f0beb34ac4a28ce0fbd7b6eb9a6039d6f9b74304a5abb3df9 155B / 155B                                                     0.3s
 => => sha256:6c2d8e2bae38638efd21358e67410e6e73f83f95eee95ad5e42d272b0effce94 110.09MB / 110.09MB                                            26.2s
 => => sha256:e4bc8fc554c31c0fb115880309eafbbdfcbeaa5259281e59b26346027eb06831 281.50kB / 281.50kB                                             0.8s
 => => sha256:803daa35ea4774c1839c77f23e37057a576d5cce3a041b2e2b5f700cf3f036b9 155B / 155B                                                     0.3s
 => => extracting sha256:e4bc8fc554c31c0fb115880309eafbbdfcbeaa5259281e59b26346027eb06831                                                      0.1s
 => => extracting sha256:803daa35ea4774c1839c77f23e37057a576d5cce3a041b2e2b5f700cf3f036b9                                                      0.0s
 => => extracting sha256:6c2d8e2bae38638efd21358e67410e6e73f83f95eee95ad5e42d272b0effce94                                                      4.9s
 => => extracting sha256:4ac7b6f906a9a07f0beb34ac4a28ce0fbd7b6eb9a6039d6f9b74304a5abb3df9                                                      0.3s
 => [linux/amd64 stage-1 1/4] FROM docker.io/library/alpine@sha256:eb3e4e175ba6d212ba1d6e04fc0782916c08e1c9d7b45892e9796141b1d379ae            1.1s
 => => resolve docker.io/library/alpine@sha256:eb3e4e175ba6d212ba1d6e04fc0782916c08e1c9d7b45892e9796141b1d379ae                                0.1s
 => => sha256:29291e31a76a7e560b9b7ad3cada56e8c18d50a96cca8a2573e4f4689d7aca77 2.81MB / 2.81MB                                                 0.9s
 => => extracting sha256:29291e31a76a7e560b9b7ad3cada56e8c18d50a96cca8a2573e4f4689d7aca77                                                      0.2s
 => [linux/arm64 builder 1/6] FROM docker.io/library/golang:alpine@sha256:8b27003087e1612c4a0ba06be640b2cfcebb5590c2de9869d88669f1a9da8286    30.5s
 => => resolve docker.io/library/golang:alpine@sha256:8b27003087e1612c4a0ba06be640b2cfcebb5590c2de9869d88669f1a9da8286                         0.1s
 => => sha256:461d32ee99a86d8cff5bd1794440f3943fd17d6a50c3eadb153b405fbf12a55b 155B / 155B                                                     0.3s
 => => sha256:9c101dd505adce3f3cb542dc55cd2884eaa419d3f5243d616d215c7b4a291246 104.32MB / 104.32MB                                            24.0s
 => => sha256:9781401cc20597a23238fc98d1ad29844622e5789f01f5bd8253a08a6862f758 155B / 155B                                                     0.8s
 => => sha256:52e4494067509f38a40cef1846fd0e7ff8752d93c2530c65c8522c39100d7353 281.68kB / 281.68kB                                             0.4s
 => => extracting sha256:52e4494067509f38a40cef1846fd0e7ff8752d93c2530c65c8522c39100d7353                                                      0.1s
 => => extracting sha256:9781401cc20597a23238fc98d1ad29844622e5789f01f5bd8253a08a6862f758                                                      0.0s
 => => extracting sha256:9c101dd505adce3f3cb542dc55cd2884eaa419d3f5243d616d215c7b4a291246                                                      5.4s
 => => extracting sha256:461d32ee99a86d8cff5bd1794440f3943fd17d6a50c3eadb153b405fbf12a55b                                                      0.0s
 => [linux/arm64 stage-1 1/4] FROM docker.io/library/alpine@sha256:eb3e4e175ba6d212ba1d6e04fc0782916c08e1c9d7b45892e9796141b1d379ae            1.0s
 => => resolve docker.io/library/alpine@sha256:eb3e4e175ba6d212ba1d6e04fc0782916c08e1c9d7b45892e9796141b1d379ae                                0.2s
 => => sha256:fd3acdcea5682abced546ec19fb6ebee725c5184e5d91614c469c0a79e67f2d0 2.71MB / 2.71MB                                                 0.8s
 => => extracting sha256:fd3acdcea5682abced546ec19fb6ebee725c5184e5d91614c469c0a79e67f2d0                                                      0.1s
 => [linux/arm/v7 stage-1 2/4] RUN mkdir /app                                                                                                  0.2s
 => [linux/arm64 stage-1 2/4] RUN mkdir /app                                                                                                   0.2s
 => [linux/amd64 stage-1 2/4] RUN mkdir /app                                                                                                   0.2s
 => [linux/arm/v7 stage-1 3/4] WORKDIR /app                                                                                                    0.1s
 => [linux/amd64 stage-1 3/4] WORKDIR /app                                                                                                     0.1s
 => [linux/arm64 stage-1 3/4] WORKDIR /app                                                                                                     0.1s
 => [linux/arm64 builder 2/6] RUN mkdir /app                                                                                                   1.4s
 => [linux/arm/v7 builder 2/6] RUN mkdir /app                                                                                                  0.5s
 => [linux/amd64 builder 2/6] RUN mkdir /app                                                                                                   0.2s
 => [linux/arm64 builder 3/6] ADD . /app/                                                                                                      0.1s
 => [linux/arm64 builder 4/6] WORKDIR /app                                                                                                     0.1s
 => [linux/arm/v7 builder 3/6] ADD . /app/                                                                                                     0.1s
 => [linux/amd64 builder 3/6] ADD . /app/                                                                                                      0.2s
 => [linux/arm64 builder 5/6] RUN go mod init example.com/m                                                                                    0.5s
 => [linux/arm/v7 builder 4/6] WORKDIR /app                                                                                                    0.1s
 => [linux/amd64 builder 4/6] WORKDIR /app                                                                                                     0.1s
 => [linux/arm/v7 builder 5/6] RUN go mod init example.com/m                                                                                   0.4s
 => [linux/amd64 builder 5/6] RUN go mod init example.com/m                                                                                    0.2s
 => [linux/amd64 builder 6/6] RUN go build -o hello .                                                                                          0.4s
 => [linux/arm64 builder 6/6] RUN go build -o hello .                                                                                          4.0s
 => [linux/arm/v7 builder 6/6] RUN go build -o hello .                                                                                         3.8s
 => [linux/amd64 stage-1 4/4] COPY --from=builder /app/hello .                                                                                 0.1s
 => [linux/arm/v7 stage-1 4/4] COPY --from=builder /app/hello .                                                                                0.1s
 => [linux/arm64 stage-1 4/4] COPY --from=builder /app/hello .
```

How does this buildx magic work, you ask? Well, behind the scenes, buildx builds three Docker images (one for each of arm, arm64, and amd64) using QEMU and binfmt_misc as needed. When it's done building, it will create a Docker manifest list which contains pointers to the three images. In other words, a "multi-arch image" is really just a manifest list with links to images built per architecture.


Now that the images for the specified architectures are built, we can either push them all to DockerHub or we can load a specific architecture to our local docker image registry and try it out

```bash
$ docker buildx build --load --platform=linux/arm64 -t hello-go-arch .    
                                                                                             
[+] Building 3.3s (17/17) FINISHED
 => [internal] load build definition from Dockerfile                                                                                           0.0s
 => => transferring dockerfile: 253B                                                                                                           0.0s
 => [internal] load .dockerignore                                                                                                              0.0s
 => => transferring context: 2B                                                                                                                0.0s
 => [internal] load metadata for docker.io/library/alpine:latest                                                                               2.1s
 => [internal] load metadata for docker.io/library/golang:alpine                                                                               1.8s
 => [builder 1/6] FROM docker.io/library/golang:alpine@sha256:8b27003087e1612c4a0ba06be640b2cfcebb5590c2de9869d88669f1a9da8286                 0.0s
 => => resolve docker.io/library/golang:alpine@sha256:8b27003087e1612c4a0ba06be640b2cfcebb5590c2de9869d88669f1a9da8286                         0.0s
 => [stage-1 1/4] FROM docker.io/library/alpine@sha256:eb3e4e175ba6d212ba1d6e04fc0782916c08e1c9d7b45892e9796141b1d379ae                        0.1s
 => => resolve docker.io/library/alpine@sha256:eb3e4e175ba6d212ba1d6e04fc0782916c08e1c9d7b45892e9796141b1d379ae                                0.1s
 => [internal] load build context                                                                                                              0.0s
 => => transferring context: 59B                                                                                                               0.0s
 => CACHED [stage-1 2/4] RUN mkdir /app                                                                                                        0.0s
 => CACHED [stage-1 3/4] WORKDIR /app                                                                                                          0.0s
 => CACHED [builder 2/6] RUN mkdir /app                                                                                                        0.0s
 => CACHED [builder 3/6] ADD . /app/                                                                                                           0.0s
 => CACHED [builder 4/6] WORKDIR /app                                                                                                          0.0s
 => CACHED [builder 5/6] RUN go mod init example.com/m                                                                                         0.0s
 => CACHED [builder 6/6] RUN go build -o hello .                                                                                               0.0s
 => CACHED [stage-1 4/4] COPY --from=builder /app/hello .                                                                                      0.0s
 => exporting to oci image format                                                                                                              1.0s
 => => exporting layers                                                                                                                        0.3s
 => => exporting manifest sha256:ad8af91a831fccf968a97c29b0a66503cc3261dffb0868e40b6aa1f158a79b4c                                              0.0s
 => => exporting config sha256:599b39d80749dd5e1e69be31d0cb7aaaea4e7721796347b22d61be376f83a931                                                0.0s
 => => sending tarball                                                                                                                         0.6s
 => importing to docker
```

And now we can run the image,

```bash
$ docker run --platform=linux/arm64 hello-go-arch                                                                                                                      
Hello, arm64!
```

Also we can try the `arm` platform

```bash
$ docker buildx build --load --platform=linux/arm -t hello-go-arch .                                                                            
$ docker run --platform=linux/arm hello-go-arch                                                                                                 
Hello, arm!
```

## Conclusion

To summarise, we saw how without making a single change to our Dockerfile we were able to build the sample application with multi-arch support, multiple docker images for arm, arm64 and amd64 were generated, and then we can load the architecture we want and run it. Its likely that in the future all of this would get automated and even more easier to use, just stuff it in in some yaml file and docker will make the required images for you, and buildx will become a standard part of docker.