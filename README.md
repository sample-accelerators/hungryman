# Hungryman

> **NOTE** This repo has been archived and the sample has been renamed to `Where for Dinner` and moved to:  
> https://github.com/vmware-tanzu/application-accelerator-samples/tree/main/where-for-dinner

## Deployment Guides

The Hungryman application can technically be deployed on almost any Cloud Native Buildpack enabled platform.  Below are detailed deployment instructions for various platforms:

* [Tanzu Application Platform](doc/TAPDeployment.md)
* Tanzu Application Services (TBD)
* Azure Spring Apps Enterprise (TBD)

## Description
The hungryman dining search solution is a multi micro-service application that implements a relatively contemporary architecture comprised of the following technologies:

* An application/API gateway (Spring Cloud Gateway)
* ReactJS Web Front End
* Reactive REST Web Services (WebFlux)
* Reactive Database Access (R2DBC)
* Asynchronous Eventing (Spring Cloud Streams and Functions)
* State (Redis)
* Security (oAuth and Tanzu AppSSO)

Along with Spring Cloud Stream, CloudEvents and KNative eventing are supported as an alternative deployment option with a small change in application dependencies (but no source code changes).  This alternative deployment method also supports KNatvie scale to zero as well as auto scaling. 

The default application configuration supports static out of the box data sets; however, is also contains options to configure alternative data sources.  Lastly, the asynchronous messaging model supports the ability to add applications to the extend functionality.

## Use Case
The value proposition of the Hungryman application is to search for availability at dining establishments.  Unlike other search applications that provide on-demand results of dining availability in real time, Hungryman attempts to ease the frustration of obtaining reservations at establishments that have limited availability (i.e. they are hard to get into).  Real time searches at these types of establishment tend to always show no availability and forces a user to either continually hit a *search* button or come back to the reservation site multiple times to check if availability has opened.  This is wear Hungryman steps in; it takes a set of search parameters such a time window, establish name, establishment type, and/or zip code and continually searches for availability on the users behalf.  The user can come back at a later time and check to see what availability the application has found.  Hungryman continues searching and updating availability results as they change.  A real time event module can also be added that sends real time notifications to a user over a configured medium such as SMS, email, or a proprietary application.

## Application Architecture

Hungryman consists of five main service modules:

* A gateway (`hungryman-api-gateway`)
* A web UI service (`hungryman-ui`)
* An API service for submitting search requests (`hungryman-search`)
* A processor service for executing searches against availability sources and managing results (`hungryman-search-proc`)
* An API service that stores and provides access to availability results (`hungryman-availability`)

Hungryman also has two optional components:

* A crawler service that collects establishment availability and provides a query API (`hungrman-crawler`)
* A real time notification service that publishes availability to configured destinations (`hungryman-notifiy`)

The high level data flow looks like the following:

![](doc/images/HungrymanHighLevelArch.png)

Note that the event channel protocols are not specified as they can be swapped out with different implementations.  By default, the application uses the RabbitMQ binding for Spring Cloud Streams, but could be swapped for any supported binding.  Because the channel end points are implements as Java `functions`, they are support `CloudEvents` and the event channels can be orchestrated through other mechanisms such as KNative eventing.  As you will see later on, there are provided deployment configurations in this repository for RabbitMQ service binding and KNative eventing.

### Spring Cloud Streams Implementation

The default build and deployment architecture uses Spring Cloud Streams as the asynchronous messaging implementation with RabbitMQ as the default binding.  The messaging topology is depicted in the following abridged architecture diagram: 

![](doc/images/SCSMessaging.png)

### KNative Eventing 

The `kneventing` branch of the Hungryman repository contains build configuration that removes Spring Cloud Streams dependencies and relies on the Spring Cloud Function webflux library for receiving events over HTTP.  KNative eventing fully embraces this paradigm and integrates seamlessly with functions exposed via the Spring Cloud Functions module.

The architecture still uses Spring Cloud Streams as the messaging implementation for initially emitting searches, and then a KNative source implementation is inserted to create a bridge between the Spring Cloud Streams binding and the downstream KNative eventing concepts.  For the RabbitMQ implementation, a KNative RabbitMQ Source is declared to read events from a RabbitMQ exchange and propagate them through KNative eventing down to the subscribing services.  The following is a diagram of the KNative event concepts used for Hungryman:

![](doc/images/KNativeEventing.png)  


