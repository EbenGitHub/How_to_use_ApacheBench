# How_to_use_ApacheBench
####
## Installation
> Refresh the package database.
```bash
apt-get update
```
> Install the apache2-utils package to get access to ApacheBench.
```bash
apt-get install apache2-utils
```
####
## Limited Privilege User
> Next, create the user that will manage Ruby. It’s not a good idea to run some of the commands in the next section as root.
```bash
useradd -m -d /home/test -s /bin/bash -g sudo test
```
> What this command accomplishes:
- useradd - create a new user
- -m - create the home directory
- -d /home/test - set the user’s home directory to /home/test
- -s /bin/bash - make the user’s default shell bash (Ubuntu uses dash by default)
- -g sudo - add user to the sudo group (for running commands with sudo)
- test - the name of the new user
> Set the password for the new user.
```bash
passwd test
```
> Switch to the new user.
```bash
su test
```
####
## RVM
> The Ruby Version Manager makes it easy to work with different Ruby environments. It takes care of the process of installing specific Ruby versions and isolating gemsets. It is currently installed by running a bash script from their website.
```bash
\curl -L https://get.rvm.io | bash -s stable
```
> In order to use the rvm command, you need to first run the rvm script.
```bash
source ~/.rvm/scripts/rvm
```
> If you want, you can put it in your .bashrc so that rvm is available any time you login as the user.
```bash
echo "source ~/.rvm/scripts/rvm" >> ~./bashrc
```
> You can verify that the rvm script is being used by checking the head of type. It should be a function and not hashed.
```bash
type rvm | head -1

rvm is a function
```
> Next, install Ruby 2.0.0. RVM will ask for the user’s password because it needs to install an assortment of dependencies before it can make Ruby. Since RVM builds Ruby from source, this step may take a while.
```bash
rvm install 2.0.0
```
> Switch to the new Ruby. This might happen by default after the installation, but checking doesn’t hurt.
```bash
rvm use 2.0.0
```
####
## Testing
> Install Sinatra. It’s a microframework/DSL for creating Ruby web applications. The --no-* flags skip the documentation.
```bash
gem install sinatra --no-rdoc --no-ri
```
> Create the sample sinatra app which just echoes “hello world”.
```bash
cd ~
vim app.rb

# app.rb
require 'sinatra'

get '/' do
  'hello world'
end
```
> Run the server.
```bash
ruby app.rb
```
> With the server finally up, you can start load testing. A call to ab looks like this:
```bash
ab -n <num_requests> -c <concurrency> <addr>:<port><path>
```
> Open another terminal and ssh into the server again. Run a test with ApacheBench. I used 1000 requests with a concurrency of 100. Don’t forget the final ‘/’ for the path.
```bash
ab -n 1000 -c 100 http://localhost:4567/

Server Software:        WEBrick/1.3.1
Server Hostname:        localhost
Server Port:            4567

Document Path:          /
Document Length:        11 bytes

Concurrency Level:      100
Time taken for tests:   3.410 seconds
Complete requests:      1000
Failed requests:        0
Write errors:           0
Total transferred:      288000 bytes
HTML transferred:       11000 bytes
Requests per second:    293.23 [#/sec] (mean)
Time per request:       341.034 [ms] (mean)
Time per request:       3.410 [ms] (mean, across all concurrent requests)
Transfer rate:          82.47 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   2.0      0      11
Processing:   185  332  90.3    311     578
Waiting:       28  280  83.2    267     574
Total:        193  333  89.7    311     578

Percentage of the requests served within a certain time (ms)
  50%    311
  66%    357
  75%    423
  80%    446
  90%    467
  95%    480
  98%    490
  99%    501
100%    578 (longest request)
```
> My results converged around 300 requests/second. WEBrick is not known for its speed. Go ahead and interrupt the server with Ctrl-c.
####
## Install Thin
> Thin is a popular ruby web server that uses Mongrel for parsing and EventMachine for non-blocking IO. Install Thin and run the server again. Sinatra should load Thin automatically and let you know (“…with backup from Thin”).
```bash
gem install thin
ruby app.rb
```
> Now, try the load test again. It should be a bit faster this time.
```bash
Server Software:        thin
Server Hostname:        localhost
Server Port:            4567

Document Path:          /
Document Length:        11 bytes

Concurrency Level:      100
Time taken for tests:   1.339 seconds
Complete requests:      1000
Failed requests:        0
Write errors:           0
Total transferred:      244000 bytes
HTML transferred:       11000 bytes
Requests per second:    747.00 [#/sec] (mean)
Time per request:       133.870 [ms] (mean)
Time per request:       1.339 [ms] (mean, across all concurrent requests)
Transfer rate:          178.00 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   1.8      0       8
Processing:    55  128  19.9    132     155
Waiting:       42  116  19.7    121     144
Total:         62  129  18.5    132     156

Percentage of the requests served within a certain time (ms)
  50%    132
  66%    135
  75%    137
  80%    139
  90%    144
  95%    149
  98%    152
  99%    155
100%    156 (longest request)
```
