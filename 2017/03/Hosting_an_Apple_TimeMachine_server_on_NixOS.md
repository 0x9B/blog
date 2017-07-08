# Hosting an Apple TimeMachine server on NixOS

My little brother just got himself a new Macbook Pro for school and he
wanted to backup his stuff using TimeMachine, so naturally I’d have to
setup the server for him... Turns out doing this on NixOS requires a bit
of extra config that’s not really well documented so I’m posting it here.
I have a NFS share that’s mounted on /mnt/timemachine as the storage backend.

```nix
networking.hostName = "timemachine";
networking.firewall.enable = false;

time.timeZone = "Asia/Hong_Kong";

services = {
  openssh.enable = true;
  avahi = {
    enable = true;
    publish = {
      enable = true;
      userServices = true;
    };
  };
  netatalk = {
    enable = true;
    volumes = {
      "timemachine" = {
        path = "/mnt/timemachine";
        "valid users" = "timemachine";
        "time machine" = "yes";
      };
    };
  };
};

users.extraUsers.timemachine = {
  isNormalUser = true;
  extraGroups = [ "wheel" ];
};
```
