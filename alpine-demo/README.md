## Alpine Bash Docker Image

This Docker image, built upon Alpine 3.19.0, offers a streamlined environment equipped with Bash. Its lightweight nature makes it an ideal choice for effortlessly executing scripts. [hub.docker.com/r/samarpanda/alpine-demo](https://hub.docker.com/r/samarpanda/alpine-demo)

### Features:

- **Efficient Script Execution:** Utilize the power of Bash within this minimalist Alpine-based Docker image to seamlessly run your scripts.
- **Optimized Performance:** Engineered for efficiency and ease of use, this container provides a nimble platform for script execution.
- **Alpine Base:** Leveraging Alpine Linux 3.19.0, this image maintains a small footprint, ensuring rapid deployment and resource efficiency.

### Usage:

Start using this image by pulling it from Docker Hub:

```bash
docker pull samarpanda/alpine-demo:latest
```

Run a container based on this image & remove after the run:

```bash
docker run --rm -it samarpanda/alpine-demo:latest
```

Delete image from local:

```bash
docker image rm -f samarpanda/alpine-demo:latest
```

Docker image build command:
```bash
docker build -t samarpanda/alpine-demo:latest -f Dockerfile .
```

Feel free to customize the `entrypoint.sh` script or mount volumes as needed for your specific applications.

## Supported tags and respective Dockerfile links

- [samarpanda/alpine-demo:1.0.2](https://github.com/samarpanda/dockerhub-image-publish/blob/master/alpine-demo/Dockerfile)

## Github source code folder reference

- [samarpanda/alpine-demo:1.0.2](https://github.com/samarpanda/dockerhub-image-publish/tree/master/alpine-demo)

## Where to file issues?

[https://github.com/samarpanda/dockerhub-image-publish/issues](https://github.com/samarpanda/dockerhub-image-publish/issues)

