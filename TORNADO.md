[![](https://cloud.githubusercontent.com/assets/1317406/12406044/32cd9916-be0f-11e5-9b18-1547f284f878.png)](http://www.synapse-wireless.com/)

# Using Tornado's ioloop for SNAPconnect's scheduler

Why would you want to do use Tornado's ioloop for the scheduler? Tornado has many built in advantages for addressing the use of sockets and connections to external devices. An instance of using Tornado with SNAPconnect would be to have a local web server running that showcases data that is being polled from the SNAP network. Another might be to actually post data to a cloud server and have an open socket for connections to be received from the cloud server while still processing efficiently all the SNAP network details. Tornado uses filedescriptors in general for addressing socket communications and on a Unix clone (Linux, BSD, etc.) it will use them for processing serial data as well. This use of the ioloop for serial data allowing SNAPconnect to perform a standard select on serial data allows for more efficient handling of incoming packets from the connected serial port.

With this version of SNAPconnect, we now use the Tornado ioloop for serial connections when configured to do so. If the application is run on a Windows system, SNAPconnect will poll the serial port as it has always done, even if you are using the Tornado ioloop.

To install Tornado on a pip-enabled machine, just run:

```bash
sudo pip install tornado
```

This example assumes that the user is already working on a system with SNAPconnect installed. While written to work with the E20 gateway from Synapse, only the connection parameters for the SNAP bridge need to be changed to work on other platforms.

So what needs to be done to use Tornado's ioloop for your SNAPconnect application?

```python
from apy import ioloop_scheduler
import tornado.ioloop

# Local system settings for E20 environment
serial_conn = snap.SERIAL_TYPE_RS232
serial_port = '/dev/snap1'
snap_addr = None   # Intrinsic address on E20

SNAPCONNECT_POLL_INTERVAL = 5 # ms

# Create the SNAP rpc function list for your inbound rpcs
snapRpcFuncs = {'updateLEDs' : self.updateLEDs}

cur_dir = os.path.dirname(__file__)

# Create SNAP Connect instance. Note: we are using TornadoWeb's scheduler.
self.snapconnect = snap.Snap(
                    license_file = os.path.join(cur_dir, 'license.dat'),
                    addr = snap_addr,
                    scheduler=ioloop_scheduler.IOLoopScheduler(),
                    funcs = self.snapRpcFuncs
                    )

# Connect to local SNAP wireless network
self.snapconnect.open_serial(serial_conn, serial_port)

# Tell the Tornado scheduler to call SNAP Connect's internal poll function. Tornado already polls asyncore.
tornado.ioloop.PeriodicCallback(self.snapconnect.poll_internals, self.SNAPCONNECT_POLL_INTERVAL).start()
```

From this point the snap connect features you would normally use operate the same way as you would if you were using the default asyncore scheduler.
