# User Authentication Based on Mouse Characteristics #

This repository hosts my code for a behavioral biometric identifier - this project aims at identifying users based on their mouse-using characteristics.

## About the Data ##
The dataset is available at [https://github.com/balabit/Mouse-Dynamics-Challenge](https://github.com/balabit/Mouse-Dynamics-Challenge), with a detailed description of the data structure and the challenge.

### Situation ###
A legit user logs in to remote servers for work via their remote desktop client, with a network monitoring device (that sits in between the remote desktop client and the remote servers) recording the user's mouse movement information.

### Concern ###
Unauthorized persons might steal a user's account and use that to log into the company's remote servers.

### Task ###
First learn the characteristics of how each legit user uses their mouse, in order to identify in future sessions whether the person that logs in to the server under one user's account is indeed that user. Naturally, the assumption here is that the typical pattern a person uses their mouse is specific and characteristic, and therefore can be used as a biometric identifier.

### Data ###
The data includes 10 users' mouse movement records from remote sessions that are known to be carried out by the respective legit user; these are the training files. It also includes mouse movement records for sessions where one certain user's account is used to logged into the remote server, potentially by an unauthorized person; these are the test files.

The above-mentioned monitoring device captures: elapsed time (in seconds, with a millisecond precision) since the start of the session as recorded by the network monitoring device, elapsed time (same precision) since the start of the session as recorded by the remote desktop client, the mouse button engaged (no button, left, middle, right, mousewheel scroll), the mouse state (move, drag, a button is pressed, button released, scroll up, scroll down), the x coordinate of the cursor on the screen, and the y coordinate of the cursor.

I discover that the data has several problems: 1) the elapsed time recorded by the monitoring device is substantially different from that by the remote desktop client (in terms of a millisecond precision), 2) a user could move their mouse outside the window of the remote desktop client; when this happens the mouse coordinates are no longer recorded and placeholder (65535, 65535) are used, 3) when scrolling the mouse wheel, the coordinates are not recorded and placeholder (0, 0) are used.

## Data Processing ##
First, I address the abovementioned problems: 1) I choose elapsed time recorded by the remote desktop client, 2) users rarely move their mice outside the remote desktop client, so I simply remove these mouse records, 3) the coordinates of the mouse movement before scrolling are available, so I assign them to the mouse scrolling records.

I then aggregate the raw mouse movements into "mouse actions". Definition of a mouse action is from one click/scroll to another:
* a complete drag, that starts with a button pressed and ends with the button released with dragging in between
* move + scroll, that includes all the moves leading to a scroll and ends with the end of scrolling
* move + left single click
* move + right single click
* move + double click, where a double click is defined as two consecutive single clicks within 5 seconds of each other (5s is the maximum double-click interval for MS Windows)
* if there is a break in time longer than 5 seconds between two consecutive "move" records, the first one ends a mouse action and the second one starts a new mouse action.

## Feature Engineering ##
### Feature Construction ###
Each mouse action has these features:
* mouse action type
* average linear speed
* maximum linear speed
* distance traveled (in pixels)
* straight line distance between the beginning and the ending position of the mouse
* efficiency: ratio of straight line distance and distance traveled
* total time of the mouse action
* sum of angles in paths
* curvature: ration of the sum of angles and distance traveled
* average angular speed

### Outlier Removal ###
* From the maximum values of the coordinates I can find the largest screen size among all users. Obviously, the longest straight-line distance a mouse can travel on a rectangle screen is its diagonal. Actions with a shortest distance longer than the largest screen's diagonal are removed.

## Modeling ##
The data could be used for various tasks such as user identification (for given mouse movement records, figure out who the user is). However, the specific goal of this challenge is user authentication: for a given remote session, one user's account is used to log in; user authentication refers to determining whether the person behind the account is indeed its legit owner. I fit a LightGBM classification model to each user's mouse behavior. Model optimization is done via randomized grid search. 

## Results ##
Public leaderboard score is 0.9075. 