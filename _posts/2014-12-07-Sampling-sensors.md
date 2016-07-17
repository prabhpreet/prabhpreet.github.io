---
layout: post
title:  "Sampling sensors"
date:   2014-12-07 01:56:17
tags: matlab, testing, sensors
---
I am currently working on a robotics project and it required a bit of quick testing. The quickest way to get a reading, other than the LCD was UART + a Matlab script. I wrote my own MATLAB script, but it required some obscure functions to get it working.

So I thought I'll share the important tidbits.

### The serial object

It's kind of like the file pointer data structure in C. It will store the port info, but you'll have to use `fopen`, `fgets` and `fclose` like manipulating files in C. Actually, the port data has been abstracted to the file. However, `fclose` is a big deal if you want other programs (such as a code uploader) to access your port.

``` matlab
s = serial('COM3') % COM3 is a port. Put whatever port your connected to!

set(s,'BaudRate', 115200); %Set your baud rate!
s.InputBufferSize = 5000; %Set the input buffer size (how much data can be stored)

fopen(s);

... %Other code omitted

while(1) %Infinite loop!
	if s.BytesAvailable > 0 % To avoid timeout and other weird things
		str = fgets(s);
		...
	end
end
fclose(s); %Now you can access the object
```

### Exception/Error handling

You'll need this part because you want to close your port and let other programs access it in case something bad happens.

``` matlab 
try
... %Your code here
catch err %Catch the damn problem
fclose(s);
end;
``` 

### Plotting

Here you can plot as usual as except you'll have to use `drawnow` to plot your data immediately and `hold on` if you're going to add data to the existing plot.

``` matlab 
figure(1);
hold on;
...
while(1)
	...
	plot(A);
	drawnow;
end
``` 

### String manipulation

Here I've used `sscanf` to manipulate the string. I think there is also a direct function to get both getting a line and scanning done at the same time. You can look up the usage easily. The code basically returns a column vector according to the format specified using C's format specifiers (but look it up, to make sure!).

My data was of a white line sensor and I was getting it's left, middle and right values hence the `L:%d M:%d R:%d`.

``` matlab 
A = sscanf(str, 'L:%d M:%d R:%d');
``` 

### All the code

Here it is, in all it's glory:
``` matlab 
clear all;
close all;

figure(1);
delay = 0.5;

s = serial('COM3');

set(s,'BaudRate', 115200);
s.InputBufferSize = 5000;

fopen(s);

input('Ready?');

try
A = [0;0;0];

while(1)
    if s.BytesAvailable > 0
    str = fgets(s);
    A = sscanf(str, 'L:%d M:%d R:%d');
    bar(A)
    axis([0 4 0 255]);
    drawnow
    A = [0;0;0];
    
    end
end;
fclose(s);
catch err
    fclose(s);
end;
``` 
