# Matchbox

[![crates.io](https://img.shields.io/crates/v/matchbox_socket.svg)](https://crates.io/crates/matchbox_socket)
![MIT/Apache 2.0](https://img.shields.io/badge/license-MIT%2FApache-blue.svg)
[![crates.io](https://img.shields.io/crates/d/matchbox_socket.svg)](https://crates.io/crates/matchbox_socket)
[![docs.rs](https://img.shields.io/docsrs/matchbox_socket)](https://docs.rs/matchbox_socket)

Painless peer-to-peer WebRTC networking for rust wasm applications.

The goal of the Matchbox project is to enable udp-like, unordered, unreliable
p2p connections in web browsers to facilitate low-latency multiplayer games.

[Introductory blog post](https://johanhelsing.studio/posts/introducing-matchbox)

**WARNING:** This project is in early stages, it will break as things are
cleaned up and moved around.

It is currently an all-in-one solution, it comes with:

- A tiny signalling server, [matchbox_server](./matchbox_server). Written in
rust, uses only a couple of megabytes of memory. Also available as a docker image.
- An example browser game, using `bevy`, `bevy_ggrs`,
[matchbox_demo](matchbox_demo)
- A socket abstraction for rust wasm, [matchbox_socket](matchbox_socket)
  - With a feature, `ggrs-socket` for providing a
  [ggrs](https://github.com/gschup/ggrs) compatible socket.

## Live demo

- 2-player demo: https://helsing.studio/box_game/
- 4-player demo: https://helsing.studio/box_game/?players=4

Open each link in a separate browser window (or machine).

When enough players have joined, you should see a couple of boxes, one of which
you can move around using the `WASD` keys.

You can open the browser console to get some rough idea about what's happening
(or not happening if that's the unfortunate case).

## How it works

WebRTC allows direct connections between peers, but in order to establish
those connections, some kind of signalling service is needed.
`matchbox_server` is such a service. Once the connections are established,
however, data will flow directly between peers, and no traffic will go through
the signalling server.

The signalling service needs to run somewhere all clients can reach it over
http or https connections. In production, this usually means the public
internet.

When a client wants to join a p2p (mesh) network, it connects to the signalling
service and provides a room id. The signalling server then notifies the peers
that have already connected about the new peer (sends a `new_peer` event).

The existing peers then send back WebRTC connection offers through the
signalling service to the new client, each of which the new client responds
with an "answer". Once the peers have enough information about each other, a
WebRTCPeerConnection is established for each peer, and an unreliable, unordered
data channel is opened.

All of this, however, is hidden from rust application code. All you will need to
do on the client side, is:

- Create a new socket, and give it a signalling server url and a room id
- Regularly poll a message loop future that processes new messages. You should
do this as often as possible, and at least once per frame. If you have an async
runtime, you can simply `.await` it. If you are using Bevy, it can be spawned as
a Bevy io task (see demo).

You will then get notified whenever a new peer data connection has been
established, and you will get all packets from peers in a single channel.
Packets include a boxed `u8` slice and the corresponding client's id.

Similarly, you can send packets to clients using a simple non-blocking method.

## Thanks!

- A huge thanks to Ernest Wong for his [Dango Tribute
experiment](https://github.com/ErnWong/dango-tribute)! `matchbox_socket` is
heavily inspired its wasm-bindgen server_socket and Matchbox would probably not
exist without it.

## License

All code in this repository dual-licensed under either:

- MIT License (LICENSE-MIT or http://opensource.org/licenses/MIT)
- Apache License, Version 2.0 (LICENSE-APACHE or http://www.apache.org/licenses/LICENSE-2.0)

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
