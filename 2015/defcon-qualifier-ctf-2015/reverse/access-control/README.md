# DEF CON Qualifier 2015: access control

**Category:** Reverse Engineering
**Points:** 1

> It's all about who you know and what you want. access_control_server_f380fcad6e9b2cdb3c73c651824222dc.quals.shallweplayaga.me:17069
>
> [Download Client](client_197010ce28dffd35bf00ffc56e3aeb9f)

## Write-up

Running the file, it requires IP, so we provide it the IP to the server. It then asks for message which (after some simple disassembling) we know is compared to "hack the world". Filling this in, we notice an interaction with the server which then gives us a list of users. We also know that grumpy is the user being used.

Initially, I wrote up a script to capture the interaction going in and out, but the password seems to be dependant on each execution, so we cannot hardcode a script. In IDA, we notice that there is a password calculation done using the username so maybe we can access other users just by sending different parameters.

"grumpy" is a small username and some of the usernames are quite long so we look for a different location to throw the username in and patch the binary. I settled for the error message location.
I overwrote "Could not create socket" with "grumpy\n" (padding with nulls at the end), and redirected the reference to "grumpy\n" to this. Next, I overwrote "connect failed. Error" with "grumpy" (again, padded with nulls) and redirected the 2 references to "grumpy" to this. Now, we can basically put what ever usernames we want into the executable and we will be logged in as that.

But first, a sanity check. Running the patched executable gives the exact same results as the original (except in the case of errors messages but we don't care about that). OK, so we're good there.

Now, we patch this with the usernames from the list we had got, one by one. This can be done manually since we have only the users mrvito, gynophage, selir, jymbolia, sirgoon, duchess, and deadwood to try. Some of these users don't even have rights to execute "list users", as it turns out, but for "duchess", we reach a challenge after which it hangs. Ah, so duchess is the admin.

Going back to IDA, we see that there seems to be some sort of a state machine type thing being held, and there's two locations where a "print key" is sent. Some analysis later, we see that the part that does receive check for "answer?" is not executed since it belongs to state 3, and for a valid log in, it jumps to state 2 later. So, we again patch one byte in the binary to make it jump to state 3 instead.

Running this then gives us the flag. `the key is: The only easy day was yesterday. 44564` is the exact message from the server.

Unfortunately, our team was too late in solving this challenge, and it was solved only after the contest ended by [Jay "f0xtr0t" Bosamiya](https://github.com/jaybosamiya/).

This challenge was supposed to be a reverse engineering challenge but turns out, by giving the client, a patching was "enough" to solve it. As far as I can tell though, no one else seems to have used this method to solve it.

A patched version of the binary can be downloaded [here](client_mod_for_flag).
