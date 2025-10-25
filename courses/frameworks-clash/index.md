---
title: Clash of Frameworks
description: A hands-on course where I build the same AI chat application using Next.js, Tanstack Start, and React SPA to compare their strengths and discover which framework works best for different scenarios.
---

Build once, compare forever. In this course, I'll create the same AI chat application using three different React frameworks and document the real-world differences.

## Target Audience

This course is for React developers at any level who want to understand the practical differences between modern React frameworks and meta-frameworks. Whether you're just starting with React frameworks, choosing a stack for your next project, or curious about how these tools compare in real-world scenarios, this course will give you hands-on insights through building the same application three different ways.

Perfect for:
- Developers new to React frameworks looking to make an informed first choice
- Experienced React developers wanting to expand their framework knowledge
- Teams evaluating which framework to adopt for their projects
- Anyone curious about Next.js, Tanstack Start, or modern SPA development

## Prerequisites

- Basic understanding of React fundamentals (components, props, state)
- Familiarity with JavaScript/TypeScript (we'll explain TypeScript concepts as we go)
- Basic knowledge of APIs (REST APIs)
- Comfort with the command line and package managers (npm/pnpm/yarn)

**Nice to have but not required:**
- Previous experience with any React framework
- Knowledge of Server-Side Rendering concepts
- Experience with AI APIs

## Learning Objectives

- Build the same AI chat application three times using different frameworks
- Understand the key differences between Next.js, Tanstack Start, and React SPA
- Learn routing strategies across different frameworks
- Compare data fetching patterns and server-side rendering approaches
- Evaluate developer experience, build times, and performance
- Make informed decisions about which framework to use for specific use cases
- Understand the trade-offs between full-stack frameworks and client-only applications

## Course Outline

### Module 1 - Building with Next.js

Learn how to build an AI chat application using Next.js, leveraging its powerful App Router, Server Components, Server Actions, and the new Cached Components feature.

**Topics Covered:**
- Setting up a Next.js 15+ project with TypeScript
- Implementing file-based routing with the App Router
- Using Server Components for optimal performance
- Utilizing Cached Components for improved performance and reduced re-renders
- Implementing streaming responses for AI chat
- Server Actions for handling form submissions and mutations
- API routes vs Server Actions - when to use what
- Caching strategies with the new `use cache` directive
- Optimizing with built-in features (Image, Font optimization)
- Deployment considerations on Vercel and other platforms

### Module 2 - Building with Tanstack Start

Explore Tanstack Start, the new full-stack React framework that brings together Tanstack Router, Query, and more into a cohesive development experience.

**Topics Covered:**
- Setting up a Tanstack Start project from scratch
- Understanding Tanstack Router and file-based routing
- Implementing loaders and actions for data fetching
- Server-side rendering with Tanstack Start
- Integrating Tanstack Query for client-side data management
- Handling real-time streaming with AI responses
- Type-safe routing and data fetching
- Deployment options and considerations

### Module 3 - Building with React SPA

Build a traditional Single Page Application using React, client-side routing, and modern state management to understand the differences from full-stack frameworks.

**Topics Covered:**
- Setting up a Vite + React project
- Implementing routing with React Router
- Client-side state management strategies
- Fetching data from external APIs
- Handling authentication in a client-only app
- Progressive Web App (PWA) considerations
- Optimizing bundle size and code splitting
- Deployment to static hosting (Netlify, Cloudflare Pages)

### Module 4 - The Comparison

A comprehensive analysis comparing all three implementations across multiple dimensions.

**Comparison Criteria:**
- Initial load performance
- Time to Interactive (TTI)
- Development experience and setup time
- Build times and bundle sizes
- Developer tooling and debugging experience
- Deployment complexity and options
- SEO capabilities
- Server costs and infrastructure needs
- Learning curve and documentation quality
- Community size and ecosystem

**Final Verdict:**
- When to use Next.js (and when not to)
- When Tanstack Start shines
- When a simple SPA is the right choice
- Hybrid approaches and future considerations

## What You'll Gain

By the end of this course, you'll have:

- **Three working applications** demonstrating the same features across different frameworks
- **Practical knowledge** of real-world trade-offs between frameworks
- **Decision-making skills** to choose the right tool for your projects
- **Hands-on experience** with modern React development patterns
- **Performance insights** backed by actual metrics and measurements
- **A reference repository** you can use for future projects

## Course Structure

Each module follows a similar pattern:
1. **Setup & Configuration** - Get the project running
2. **Core Features** - Build the main functionality
3. **Optimization** - Improve performance and user experience
4. **Deployment** - Get it live on the web
5. **Reflection** - Document observations and gotchas

The final comparison module brings everything together with data, metrics, and recommendations.

---

*Note: This course is framework-agnostic in its teaching approach - it's not about advocating for one tool over another, but helping you understand the real differences so you can make informed decisions.*
