# firewallctl
Safely deploy script based firewalls in Linux.

## Usage

<pre>
$ firewallctl 
usage: firewallctl [start|configure|confirm|status]
</pre>

<dl>
  <dt>start</dt>
  <dd>For compatibility with init scripts. If there a last-known-good firewall, it will be applied immediately. If there isn't a known-good firewall firewallctl attempts to run the firewall script and then mark it as known-good. <strong>This bypasses the rollback timeout, so should not be used normally.</strong></dd>

  <dt>configure [timeout]</dt>
  <dd>The main way to use <tt>firewallctl</tt>. Runs <em>status</em> then asks for confirmation. If confirmed, applies the changed firewall and starts the rollback timeout (defaults to 120 seconds if not specified).</dd>
  
  <dt>confirm</dt>
  <dd>Once a configuration has been applied, it must be confirmed within <em>timeout</em> seconds using the <em>confirm</em> command, or the last known good firewall will be re-applied (and a log message will be printed to syslog to indicate this).</dd>
  
  <dt>status</dt>
  <dd>Just output the current state of the firewall (e.g. whether a new firewall is waiting to be applied).</dd>
</dl>

## Example usage

<pre>
test-firewall:/etc/network# firewallctl configure 10
A new firewall is waiting to be deployed.

--- /etc/network/last-known-good-firewall   2016-07-21 11:09:41.953415248 +1200
+++ /etc/network/firewall   2016-07-21 11:09:58.861235524 +1200
@@ -1869,6 +1869,7 @@
     echo "-A Demo -s 198.51.100.0 -j ACCEPT"
     echo "-A Demo -s 198.51.100.1 -j ACCEPT"
     echo "-A Demo -s 198.51.100.2 -j ACCEPT"
+    echo "-A Demo -d 192.0.2.1 -j DROP"
     # 
     # Next up we frobnicate some packets
     # As per change control #1234

Apply this change? [y/N]y

Applied firewall. Use `firewallctl confirm' within 10 seconds to confirm.
test-firewall:/etc/network#
[... time passes ...]
test-firewall:/etc/network# grep firewallctl /var/log/syslog
Jul 21 11:10:26 test-firewall root: firewallctl: rolled back firewall!
test-firewall:/etc/network# firewallctl configure 30
A new firewall is waiting to be deployed.

--- /etc/network/last-known-good-firewall   2016-07-21 11:09:41.953415248 +1200
+++ /etc/network/firewall   2016-07-21 11:09:58.861235524 +1200
@@ -1869,6 +1869,7 @@
     echo "-A Demo -s 198.51.100.0 -j ACCEPT"
     echo "-A Demo -s 198.51.100.1 -j ACCEPT"
     echo "-A Demo -s 198.51.100.2 -j ACCEPT"
+    echo "-A Demo -d 192.0.2.1 -j DROP"
     # 
     # Next up we frobnicate some packets
     # As per change control #1234

Apply this change? [y/N]y

Applied firewall. Use `firewallctl confirm' within 30 seconds to confirm.
test-firewall:/etc/network# firewallctl confirm
Confirmed.
test-firewall:/etc/network# firewallctl status
Running firewall is up to date.
</pre>
