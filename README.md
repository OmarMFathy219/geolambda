# GeoLambda: geospatial AWS Lambda Layer

The GeoLambda project provides public Docker images and AWS Lambda Layers containing common geospatial native libraries. GeoLambda contains the libraries PROJ.5, GEOS, GeoTIFF, HDF4/5, SZIP, NetCDF, OpenJPEG, WEBP, ZSTD, and GDAL. For some applications you may wish to minimize the size of the libraries by excluding unused libraries, or you may wish to add other libraries. In this case this repository can be used as a template to create your own Docker image or Lambda Layer.

## Usage

While GeoLambda was initially intended for AWS Lambda they are also useful as base geospatial Docker images.

### Lambda Layer

To use one of the public GeoLambda layers you will need the ARN for the layer in same region as your Lambda function. Currently, GeoLambda layers are available in `us-east-1`, `us-west-2`, and `eu-central-1`. If you want to use it in another region please file an issue or you can also create your own layer using this repository.

#### v1.1.0

| Region | ARN |
| ------ | --- |
| us-east-1 | |
| us-west-2 | |
| eu-central-1 | |

### Docker images

The Docker images used to create the Lambda layer are also published to Docker Hub, and thus are also suitable for general use as a base image for geospatial applications. 

The developmentseed/geolambda image in Docker Hub is tagged by version.

	$ docker pull developmentseed/geolambda:<version>

| geolambda | GDAL  |
| -------- | ----  |
| 1.0.0    | 2.3.1 |
| 1.1.0    | 2.4.0 |

Or just include it in your own Dockerfile as the base image.

```
FROM developmentseed/geolambda:${VERSION}
```

The GeoLambda image does not have an entrypoint defined, so a command must be provided when you run it. This example will mount the current directory to /work and run the container interactively.

	$ docker run --rm -v $PWD:/work -it developmentseed/geolambda:latest /bin/bash


## Creating a new Lambda Layer



```
$ docker build . -t mygeolambda:latest
```

Then, package up the resulting files by using the [provided script](bin/package.sh). 

```
$ docker run -t mygeolambda:latest
```

## Development

Contributions to the geolambda project are encouraged. The goal is to provide a turnkey method for developing and deploying geospatial Python based projects to Amazon Web Services. The 'master' branch in this repository contains the current state as deployed to the `developmentseed/geolambda:latest` image on Dockerhub, along with a tag of the version. The 'develop' branch is the development version and is not deployed to Dockerhub.

When making a merge to the `master` branch be sure to increment the `VERSION` in the CircleCI config.yml file. Circle will push the new version as a tag to GitHub and build and push the image to Docker Hub. If a GitHub tag already exists with that version the process will fail.

### Create a new version

Use the Dockerfile here as a template for your own version of GeoLambda. Simply edit it to remove or add additional libraries, then build and tag with your own name. The steps below are used to create a new official version of GeoLambda, replace "developmentseed/geolambda" with your own name.

To create a new version of GeoLambda follow these steps. 

- update the version in the `VERSION` file
- build the image:
  
```
$ VERSION=$(cat VERSION)
$ docker build . -t developmentseed/geolambda:${VERSION}
```

- Push the image to Docker Hub:

```
$ docker push developmentseed/geolambda:${VERSION}
```

- Create deployment package:

```
$ docker run --entrypoint package.sh --rm -v $PWD:/home/geolambda -it developmentseed/geolambda:${VERSION}
```

- Push as Lambda layer

```
$ 
```

- Make layer public (if this is a newly created layer)

```
$ aws lambda add-layer-version-permission --layer-name geolambda \
	--statement-id public --version-number 1 --principal '*' \
	--action lambda:GetLayerVersion
```
