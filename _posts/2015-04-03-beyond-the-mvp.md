---
layout: post
title:  "Beyond the MVP"
author: Scott Lowe
date:   2015-04-03 15:26:10
categories: scalability
featured_image: /images/build-mvp.gif
---

Over the last few months, Max, Tim & I have really convinced ourselves that Castle is an idea worth pursuing. We're getting great feedback from the market, and we've even managed to convince a few others with deep pockets that the idea is worth pursuing too. The team is pretty confident we've come across a great business opportunity, so what now?

Easy, we need to grow–after all, [growth is the defining feature of a startup][pg-growth]. There's a motto in the startup world that goes "nail it, scale it, sail it", and that's exactly what we'll do. But in order to scale, we'll first have to re-invent ourselves.

See, we started out intending to build a minimum viable product, or MVP. The goal of an MVP is to build the bare minimum needed to test a hypothesis. Now with our hypothesis validated, we must shift our attention from build speed to scalability.

We'll do that by taking a much more modular approach to the components we build–leveraging [Browserify][browserify] to share modules between angular clients and node servers–because modularity provides flexibility, and flexibility is critical for scalability.  In addition, we'll be investing heavily in automated testing and test-driven development. Together, these approaches will allow us to build complex systems without compromising on the gauruntee of quality.

**Only then will we be able to scale.**

[pg-growth]:   http://www.paulgraham.com/growth.html
[browserify]:  http://browserify.org/
