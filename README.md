# Qubes-VM-hardening

Fend off malware at VM startup: Lock-down and remove scripts in /rw private storage that affect the execution environment.
Leverage Qubes template non-persistence to enhance the guest operating system's own defenses.
   

## vm-boot-protect.service
   * Acts at VM startup before private volume /rw mounts
   * User: Protect /home executable files as immutable
   * Root: Quarantine all /rw configs & scripts, with whitelisting
   * Re-deploy custom or default files to /rw on each boot
   * SHA256 checksumming guards against unwanted changes
   * Provides rescue shell on error or request


## Installing

1. In a template VM, install the service files
   ```
   cd Qubes-VM-hardening
   sudo sh ./install
   ```

2. Activate by specifying one of the following Qubes services for your VM(s)...
   - `vm-boot-protect` - Protects executables/scripts within /home/user and may be used with wide array of Qubes VMs including standalone, appVMs, netVMs, Whonix, etc.
   - `vm-boot-protect-root` -  Protects /home/user as above, automatic /rw executable deactivation, whitelisting, checksumming, deployment. Works with appVMs, netVMs, etc. that are _template-based_.

   CAUTION: The -root option by default **removes** dirs /rw/config, /rw/usrlocal and /rw/bind-dirs!

3. Disabling the Qubes default passwordless-root is necessary for the above measures to work effectively. Here are two recommended ways (choose one):
   - [Enabling auth prompt for sudo](https://www.qubes-os.org/doc/vm-sudo/#replacing-password-less-root-access-with-dom0-user-prompt) configures a simple yes/no prompt that appears in dom0.
   - Uninstall the `qubes-core-agent-passwordless-root` package from the template. After doing this, you will have to use `qvm-run -u root` from dom0 to run any VM commands as root.
   
---

### Usage

   Operation is automatic and will result in either a normal boot process with full access to the private volume at /rw, or a rescue service mode providing an xterm shell and the private volume quarantined at /dev/badxvdb.

   At the `vm-boot-protect` level, certain executable files in /home will be made immutable so PATH and `alias` cannot be used to hijack commands like `su` and `sudo`, nor can impostor apps autostart whenever a VM starts. This prevents normal-privilege attacks from gaining persistence at startup. 

   At the `vm-boot-protect-root` level, the $privdirs paths will be renamed as backups, effectively removing them from the VM startup. Then whitelisting, hash/checksumming and deployment are done (if configured). This protects VM startup from attacks that had previously achieved privilege escalation.

   The special `vm-boot-protect-cli` level unconditionally runs an xterm rescue shell.


### Configuration

   Files can be added to /etc/default/vms in the template to enable the following features...

   **Hashes/Checksums** are checked in ../vms/vms.all.SHA and ../vms/$vmname.SHA files. File paths contained in them must be absolute. See man page for `sha256sum -c`.

   **Whitelists** are checked in ../vms/vms.all.whitelist and ../vms/$vmname.whitelist files, and file paths contained in them must start with `/rw/`. A default is provided in ..vms/sys-net.whitelist to preserve Network Manager connections and sleep module list in sys-net.

   **Deployment** files are copied _recursively_ from ../vms/vms.all/rw/ and ../vms/$vmname/rw/ dirs. Example is to place the .bashrc file in /etc/default/vms/vms.all/rw/home/user/.bashrc .


### Scope and Limitations

   The *vm-boot-protect* concept enhances the guest operating system's own defenses by using the *root volume non-persistence* provided by the Qubes template system; thus a relatively pristine startup state may be achieved if the *private* volume is brought online in a controlled manner. Protecting the init/autostart files should result in Qubes template-based VMs that boot 'cleanly' with much less chance of being affected by malware initially. Even if malware persists in a VM, it should be possible to run other apps and terminals without interference if the malware has not escalated to root (admittedly, a big 'if').

   Conversely, attacks which damage/exploit the Ext4 private filesystem itself or quickly re-exploit network vulnerabilities could conceivably still persist at startup. Further, repeated running of some apps such as Firefox, Chrome, LibreOffice, PDF viewers, online games, etc. may allow malware to persist in a VM; this is not only because of the complexity of the formats handled by such apps, but also because of settings contained in javascript or which specify commands to be executed by the app. Therefore, setting apps to autostart can diminish protection of the startup environment.
   
   Note that as vulnerabilities are patched via system updates, malware that relies on them can only continue to persist via the kind of execution loopholes that *vm-boot-protect* closes.


### Notes

   * The service name has been changed from `vm-sudo-protect` in pre-release to `vm-boot-protect`. The install script will automatically try to disable the old service.

   * All the user-writable startup files in /home should be protected by the immutable flag; See issue #9 if you notice an omission or other problem. An extra step of disabling the flag using `sudo chattr -i` whenever the user wants to modify these startup files.

   * Adding /home or subdirs of it to $privdirs is possible. This would quarantine everything in that dir to set the stage for applying whitelists on /home contents. The $privdirs variable can be changed via the service file, for example adding a .conf file in /lib/systemd/system/vm-boot-protect.d.
   
 
## Releases
   - v0.8.1  Working rescue shell. Add sys-net whitelist, fixes.
   - v0.8.0  Adds protection to /rw, file SHA checksums, whitelists, deployment
   - v0.2.0  Protects /home/user files and dirs


