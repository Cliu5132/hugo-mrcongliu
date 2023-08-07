---
title: "Build a GatsbyJS Blog"
date: 2023-08-07
---

# Background: Why use GatsbyJS instead of ReactJS, or NextJS?

I am not the biggest fan of GatsbyJS. The reason is that it involves a lot of configuration and setup. It's one of those libraries that feels like you're learning something completely new instead of just writing Javascript. Gatsby is all about the "GastbyJS way", but it is still fun to learn building a blog using a completely unfamiliar technology.

Before we continue to the rest of this post, you should already know the basic of programming and ReactJS in order to full understand the context.

## Compare ReactJS, NextJS, and GatsbyJS

GatsbyJS is a framework written in React. It has already combined together some of the best React ecosystem libraries into one bundle and as an opinionated way about how you should use it in order to build a static website. To decide whether Gatsby is the right tool for us to build something, it is important to understand its competing technologies, like React or NextJS.

### ReactJS

React is a front-end based UI library. We write React code, which is a combination of JavaScript and JSX. The React code will then be compiled down into HTML, CSS, and JS using a loader like Webpack or Babble. Those HTML, CSS, and JS files are correct format that browsers can comsume. Therefore, users can see our website.

{{<mermaid>}}
flowchart LR

    A(React Code) --> | Compiled | B(HTML)

    A --> | Compiled | C(CSS)

    A --> | Compiled | D(JavaScript)

    B --> E(Browser)
    
    C --> E

    D --> E

{{</mermaid>}}

Create-react-app is a super fast way to start a React application and it leverages client-side Rendering.
Server-side Rendering leverages the browser to build out the HTML files whenever a user navigates to any page on our React application.
If a user goes to the /home route, then the Home React component will be accessed, causing a request sent to the browser. The browser will pull in our react code and build up the HTML file for the homepage.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/d29951b6-0ceb-ceee-5bb9-619ef2b0e19c.png)

This method requires the resource from user's local browser, which may be problematic when there are too much resource for the browser to handle. And that brings in another technology called NextJS.

### NextJS

NextJS is a tool that allows us to write React Code while taking advantage of Server-side Rendering.

Instead of asking the browser to build up HTML files for user, NextJS asks the back-end server to do that. If the user goes to /collections route, our back-end server will build the corresponding HTML file and send it to the browser. This way, we can provide high availability because the server can serve more resource to more users.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/65b4dff8-67c5-616a-0eaa-b9ba011941ff.png)

What ReactJS and NextJS have in common is that they are both dynamically generating HTML files. In contrust, GatsbyJS is a static website generator. Which means it will build up all the webpages at the end of the build step. In other words, every single webpage would have been created in our code base. Neither browser nor server needs to build any HTML file.

### GatsbyJS

Gatsby is ideal for uess cases that don't require changes in views, such as, a blog.