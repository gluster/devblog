---
layout: post
title: Gluster Heal internals
author: rkothiya
---


# Understanding the Healing internals.

### What is Self Healing ?
In case of replicated volume if one of the bricks goes offline for some time then when it comes up again self healing is required. Because both the bricks should contain the same data but as one brick was down it could have missed data that was written on the other brick while it was down.

### Processes : 

* Glusterfsd is the brick process which is running for every brick on that system. 
* This brick process (glusterfsd) has the following threads running in it : 


![](https://i.imgur.com/jK4NRxo.png)


### Lets talk about some important threads :

#### One of the important thread is rpcsvc_request_handler() which does the following : 

> 1. Initialize some variables 
> 2. while(1)
>
>    a. Take mutex lock on queue→queue_lock
>
>    b. Do some sanity checks and process the queue 
>
>    c. Unlock queue→queue_lock
>
>    d. for_each_rpc_request()
>
>            i. delete the request from list
>
>            ii. do some sanity checks
>
>            iii. Find a actor who can take action on this requests I.e.
>                 Find a function who can serve this request :
>
>                     actor = rpcsvc_program_actor(req);
>
>            iv. Serve the request by calling
>
>                     ret = actor->actor(req);
>
> 	        v. Check the return value and reply error if required
>

rpcsvc_program_actor is a helper functions. The main thing that this function does is : 

	actor = &program->actors[req->procnum];

The variable “program” has a list of actors and giving the correct procedure number gives us the correct actor. 



