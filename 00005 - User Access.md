# User Access

## Deployment Models

I envision a couple of standard ways to run Psyche:

1. **Cloud Deployment**: Running on a cloud provider (AWS, DigitalOcean, etc.) where you get a machine or VM with a public IP address. You can point a domain name to it and give people that address to log in and start using it.

2. **Local Virtualization**: Running Self/Psyche in a virtual machine on your laptop, without replacing your existing OS. You then give access to the image running on your laptop to others in your team, perhaps using an overlay network like Tailscale.

The common denominator is that the user brings their own client.

## Deployment Non-Goals

We're not pursuing a Linux distro model where you install Psyche on your notebook as your primary OS. There are obvious reasons for this:
1. It would require a lot of work to provide functionality that systems like Ubuntu already offer.
2. The end result would be similar to running a VM or browser session full-screen on an existing OS.

## Client-Server Model

The most pragmatic model is to use everyone's existing devices and available terminal software. We use the web browser because it's universal and provides the graphics capabilities we need. If we didn't need graphics, we could use standard Unix terminals, which are also available everwhere, but browsers are better.

## Current Implementation

The first iteration of Psyche runs through a series of connected components:
1. Self opens a window using X
2. Xvnc turns that into a VNC session
3. A wrapper exposes that to WebSockets
4. noVNC allows people to access the Morphic desktop through their web browser
5. Caddy or another reverse proxy handles HTTPS certificates

## Long-Term Goals

The long-term goal is to strip all of this out and have the Self world communicate directly via WebSockets to a suitable client in the browser. This would:
1. Remove complexity from the server side
2. Allow more rendering to happen on the client
3. Provide better fonts and vector graphics
4. Potentially offload some processing to the client

While it would be nice if there were a JavaScript X server to bridge the gap, that doesn't seem to exist, and with the shift to Wayland, it's unlikely to emerge.

For now, we have a stable interim solution that allows users to access shared Self worlds through their web browsers, with the only requirement being a web browser for the client and a virtual machine manager for running the world.

