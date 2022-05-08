# What is this?
This repository will hold information about my NetDot game protocol, and links to any of official client/server implementations I create (possibly community made ones in the future). I may also place the smaller/sillier clients here instead of giving them their own repository, as they really don't deserve one, but should be here for historical value at least.

Currently, these are the implementations available:
| Name/Repo | Is a Server | Is a Client | Newest Version | Earliest Version |
| --------- | ----------- | ----------- | -------------- | ---------------- |
| [NetDot (Java)](https://gitlab.com/Magicrafter13/NetDot-Java) | :heavy_check_mark: | :heavy_check_mark: | 2.0 | 1.0 |
| Bashdot | :x: | :heavy_check_mark: | 1.6 | 1.6 |
| [NetDot for Android](https://gitlab.com/Magicrafter13/NetDot-Android) | :heavy_check_mark: | :heavy_check_mark: | 2.0 | 2.0 |

# What is NetDot?
NetDot is a multiplayer (networked) version of the classic [Dots and Boxes](https://en.wikipedia.org/wiki/Dots_and_Boxes) game. This originally started as a (final?) project for the CSE 223 class at Clark college. The assignment only required us to modify our existing 2 player dots n' boxes game, to work over a network instead of 2 people using the same window. I took it a step further... Adding features like name/color change, modifiable grid size (assignment only needed 8x8), removing the player limit, and even a chat window! The latest version of this original school Java implementation can be found at [NetDot-Java](https://gitlab.com/Magicrafter13/NetDot-Java).

# Why?
Depending on what you're asking, the answer is: so I can consolidate my various code bits together, and also so I can formalize the "spec" so to speak. I designed my communications to be flexible, both so I could update it easily, and so I could make other clients in the future, or possibly even have other people make their own.

It started with the Java original, then towards the end of the quarter I added a bash client (no bash server), which was compatible with the 1.x protocol (never got updated to 2, and I lost interest). I began work on a C client, using ncurses, which was going to be much nicer than the bash implementation, but I never did anything after finishing the socket/thread programming. And most recently at the time of writing I began work on an Android version (both server and client), as the final project for my CS 458 class at Washington State University.

# So how do I play?
Well there are a few ways.

You could use my original Java version, which should easily run on Linux, macOS, or Windows - really, anything with Java (yes you too Open-BSD user). While not the prettiest version, and I have found occasional (very minor) bugs, it still works quite well all things considered.

For a bit of fun, you could use the bash version, but you'll need someone to host a server, and it is only compatible with protocol 1.x, so you'll likely need someone running an older version of my Java code.

At this point though, I'd recommend the Android version if you have an Android device. It has near perfect feature parity with the Java version (and even a couple features of its own).

See the table at the top of the document for links.

# What if I want to create my own server and/or client?
Please see my official specification on how a NetDot server/client should operate. Which can currently be found in the [spec draft](draft-spec.md).