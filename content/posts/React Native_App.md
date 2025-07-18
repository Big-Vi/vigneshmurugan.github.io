---
title: "Building My First Offline-First React Native App"
slug: "how-i-built-my-first-offline-react-native-application"
date: 2022-01-05
updated: 2025-07-19
tags:
  - reactnative
  - frontend
image: "/images/reactnative.png"
---

In 2021, I accomplished a long-standing goal: building and launching my first React Native application. The project was more than just a technical exercise; it was an opportunity to create a product that people would find genuinely useful. The result is **Resume Builder**, an offline-first mobile app now available on the [iOS App Store](https://apps.apple.com/nz/app/resume-builder/id1589350404).

This post breaks down the key technologies I used—Realm, Redux Toolkit, and TypeScript—and the core features that make the app work.

## The Tech Stack

Choosing the right tools was critical. I needed a stack that would allow for robust offline capabilities, efficient state management, and scalable, bug-free code.

### Why Realm for Offline-First?

For an offline-first application, data persistence is paramount. I chose [Realm](https://realm.io) because it’s a powerful mobile database that stores data directly on the device. This approach enables fast, responsive apps that work seamlessly without an internet connection.

While I've only scratched the surface of its capabilities, Realm offers powerful features for future expansion:
-   **Realm Sync:** This is a standout feature that synchronizes the local device database with a MongoDB Atlas instance in the cloud. It acts as a seamless bridge, enabling multi-platform applications. For instance, I could build a web version of the Resume Builder that uses MongoDB Atlas as its backend, and any changes made on the web would automatically sync back to the mobile app, and vice-versa.
-   **Realm Authentication & Functions:** These features can further simplify development by handling user management and server-side logic.

### Redux Toolkit for State Management

I initially started with vanilla Redux for state management. However, as the application grew, I found myself writing a lot of repetitive boilerplate code for actions and reducers. Switching to **Redux Toolkit** was a game-changer. It simplifies the process significantly, allowing for more concise and maintainable state management logic.

### TypeScript for Code Quality

Adopting TypeScript was a deliberate choice to improve code quality. While there was a learning curve, the benefits of a statically-typed language quickly became apparent. TypeScript helped me catch potential bugs during development, long before they could become runtime errors, leading to a more stable and reliable application.

## Essential Libraries

The React Native ecosystem is rich with libraries that accelerate development. These were some of the key modules that helped me build essential features:

-   **Drag & Drop:** [react-native-drax](https://github.com/nuclearpasta/react-native-drax)
-   **HTML to PDF Conversion:** [react-native-html-to-pdf](https://github.com/christopherdro/react-native-html-to-pdf)
-   **PDF Viewer:** [react-native-PDFView](https://github.com/rumax/react-native-PDFView)

## Key Features

Here are some of the core functionalities of the Resume Builder app:

-   **Multiple Templates:** Start with one of three professional templates, with more on the way.
-   **Live Preview:** See your changes in real-time as you build your resume.
-   **Save and Share:** Export the final resume as a PDF to your device or any cloud storage service.
-   **Drag & Drop Sections:** Easily reorder resume sections to fit your needs.
-   **Customization:** Adjust the look and feel to match your personal style.


{{< figure
  src="/images/reactnative.png"
  alt="A preview of the Resume Builder app's interface"
  caption="A preview of the Resume Builder app's interface"
  class="react-native-dashboard"
>}}

## Video Preview

Here's a quick demo of the app in action:

{{< iframe width="560" height="315" src="https://www.youtube.com/embed/Ghpi8-vjYaU" frameborder="0" style="border:0" >}}
