# Task description

## Annotation

The aim of the task is to create a multithread server for TCP/IP communication and implement a communication protocol according to a given specification. Attention, implementation of the client part is not part of the task! The client part is realized by the test environment.

## Assignment

Create a server for automatic control of remote robots. The robots themselves apply to the server and guide them to the center of the coordinate system. For testing, each robot starts at random coordinates and tries to reach the coordinate [0.0]. The robot must pick up the secret at the target coordinate. On the way to the goal, robots may come across obstacles that they have to bypass. The server can navigate multiple robots at a time and implement the communication protocol flawlessly.

## Detailed specification

Communication between the server and robots is realized by a fully text protocol. Each command ends with a pair of special symbols "\a\b". (They are two characters '\a' and '\b'.

## Server Messages:

# Job title: Multithreaded server for TCP/IP
