# 13: Scalable app example

## Overview

The CanCan sample application is a simplified video-sharing service that demonstrates several features that you can use as models for your own applications. For example, here are a few things you can learn by exploring the CanCan sample application:

-   How to build a scalable application by splitting content into fragments for upload and storage then using queries to retrieve and reassemble the fragments for efficient streaming.

-   How to configure interoperability for an application that uses canisters written in different backend languages.

-   How to implement a basic authentication model for storing videos uploaded by different users.

-   How to build a frontend that implements more sophisticated user interface features for desktop or mobile apps.

## Splitting uploaded content into multiple canisters

Because canisters are compiled WebAssembly modules, they have certain known limitations. For example, currently WebAssembly modules have a maximum of 4GB for memory and an upper limit on the number of object calls allowed.

For a video-sharing sample applications like CanCan, these limitations mean that multiple canisters are required and that data must be broken into smaller chunks for storage and retrieval.

### Implementing a distributed hash table

The initial attempt to build a scalable video-sharing service for the Internet Computer used a distributed hash table (DHT) as a backend service with simple get and put functions that distributed data—chunks of the video to be uploaded or streamed—into a predefined set of canisters. In the early phases of the project, this approach was sufficient for a proof-of-concept and verifying that the video data could be properly transcoded for storage and retrieval.

However, the scalability of the application was limited because the distributed hash table relied on a specific number of canisters that it could populate with data for storage and retrieve data from for viewing. In addition, the original implementation of the distributed hash table backend service included code to accommodate common network connectivity issues that could cause nodes to be unavailable or lose data.

## Simplifying scalability

Because the Internet Computer protocol relies on replicated state across nodes in a subnet, it provides certain guarantees about fault tolerance and failover natively that are not generally available to applications running on other platforms or protocols.

With the realization that the Internet Computer could ensure that canisters were available to receive and respond to requests,the original distributed hash table backend service was replaced with a simpler but more scalable backend service called BigMap.

BigMap provides a simple, plug-in library for building scalable applications using key-value storage on the Internet Computer. By using the BigMap library as a backend service, the CanCan sample application can dynamically chunk, serialize, and distribute data to multiple canisters.

The library offers building blocks for application-specific, in-memory data abstractions that scale using any number of canisters. Each canister still has limited capacity, but the application instantiates the canisters it needs and keeps track of the fragments that make up the full video content for each user’s videos in an index file called the `manifest`.

The code required for the `BigMap` service is much simpler than a traditional distributed hash table because the Internet Computer as a platform provides scalability, replication, failover, and fault tolerance.

## Demonstrating interoperability

The BigMap service included in the CanCan sample application repository is written in the Rust programming language. However, the CanCan sample application also demonstrates interoperability between canisters written in different languages.

In this case, the `BigMap` functionality is implemented using the Rust programming language and other services—such as encoding and decoding of video content and the management of user principals for authentication—are implemented using Motoko.

By deploying different parts of the sample application as canisters, the interaction between them provides a seamless user experience.

Although the `BigMap` service you see in the CanCan repository is written in Rust, the service was actually implemented in both Rust and Motoko programming languages to demonstrate the following:

-   You can run both Motoko code and Rust code deployed as canisters on the Internet Computer.

-   You can switch between backend languages without affecting the operation of the CanCan sample application.

-   Both language implementation work seamlessly because the Candid language provides a common language for describing the BigMap API, independent of JavaScript, Rust, or Motoko.

## Authentication model

Much like the LinkedUp sample application, the CanCan sample application uses the public-private key pair, browser-based local storage, and the `Principal` data type to authenticate users.

## Implementing frontend features

The CanCan sample application uses the React library in combination with TypeScript to implement frontend user interface.

### Data model overview

The application stores information about users and information about videos. To support most browsers, the videos are serialized into byte arrays with video data stored in 500kb segments of bytes that are referred to as [Uint8Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array) objects. When a video is requested, the manifest loads the list of chunks required to play the video and concatenates the chunks before displaying the video in a standard `<video>` element.

### User profiles

User profiles are stored as `profiles/{username}` and are defined using the following data model:

    export interface Profile {
      username: string;         // maya
      following: string[];      // [`alice`, `bob]`
      followers: string[];      // ['palice']
      uploadedVideos: string[]; // ['profiles/maya/videos/a5b54646-2ea3-4e0e-82d1-da3ab8148df2']
      likedVideos: string[];    // ['profiles/bob/videos/b74e4eb0-dea8-4a4a-a1ae-d4593dc86930']
      avatar?: string;          // ?ImageData (TODO)
    }

### Videos

Uploaded videos are identified by a unique identifier stored at `profiles/{username}/videos/{videoId}` and in `public/videos` (an array of all existing videos on the platform).

Metadata for videos is stored at `profiles/{username}/videos/{videoId}/metadata` Individual video fragments are stored at `profiles/{username}/videos/{videoId}/chunks/chunk.{0-10}`.

Videos are stored as `profiles/{username}` and are defined using the following data model:

    export interface Video {
      src: string;       // 'profiles/maya/videos/f5a44646-2ea3-4e0f-83d2-da3ab8148df2'
      userId: string;    // 'maya'
      createdAt: string; // Date.now()
      caption: string;   // 'cool movie, punk'
      tags: string[];    // ['outside', 'grilling', 'beveragino']
      likes: string[];    // ['sam', 'kelly']
      viewCount: number; // 102
      name: string; // 'grilling'
    }
