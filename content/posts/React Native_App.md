---
title: "How i built my first offline React Native application"
slug: "how-i-built-my-first-offline-react-native-application"
date: 2022-01-05
tags:
  - reactnative
  - frontend
image: "/images/reactnative.png"
---

Creating React Native app is a long-time goal that I made possible in 2021. Apart from gaining my knowledge in Redux, Realm.io & Typescript, I wanted to have my product out there in the market where people enjoy using it.

Resume builder is an offline-first React Native app I built and deployed to the [iOS app store](https://apps.apple.com/nz/app/resume-builder/id1589350404). It's built using Realm.io(Mobile database), Redux toolkit & Typescript.

## Why Realm.io?

[Realm](https://realm.io) persists the data locally which helps to develop faster and offline apps. There're plenty of features Realm offers that would take this app to next level. I've just only scratched the surface here. 

For example, the following features I could make use of:

Realm authentication, Realm sync & Functions.

With Realm sync, I would be able to extend this app further by creating a web version of this app. What Realm sync does is that it would access the locally stored data and sync it with MongoDB Atlas in the cloud. Basically, act as a middleman between Realm(Local database) & MongoDB Atlas(cloud database). 

The web version of this app could use any front-end technology & use the MongoDB Atlas as the backend. Data changed from the web app would sync back to the local Realm database in Mobile. This real-time sync is more powerful in creating web & mobile applications.

## Redux toolkit

I started with Redux for state management. Once the application starts growing, I felt like I'm doing a repetition of steps like creating reducers. Redux toolkit makes this process very easy. 

## Typescript

Initially writing typescript code was time-consuming. but once I get the hang of it, I started to see the real benefit of statically typed language over dynamically typed language(Javascript).

Typescript enabled me to code bug-free.

## React Native ecosystem

The following modules helped me to achieve the functionalities I required to launch this app thanks to React Native ecosystem.

[Drag & Drop](https://github.com/nuclearpasta/react-native-drax)

[HTML to PDF](https://github.com/christopherdro/react-native-html-to-pdf)

[PDFViewer](https://github.com/rumax/react-native-PDFView)


## Key features:

1 - Three templates to start with but more are on the way.

2 - Preview feature

3 - Save PDF format to your device or save it in the cloud.

4 - Drag & Drop feature allows you to arrange the section of the resume to your needs.

5 - Customiser enables you to change the look & feel of the resume.


![image](/images/reactnative.png)

## Preview

{{< iframe width="560" height="315" src="https://www.youtube.com/embed/Ghpi8-vjYaU" frameborder="0" style="border:0" >}}
