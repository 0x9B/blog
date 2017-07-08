# Hosting WordPress on NixOS

NixOS’ “magical” configuration never ceases to amaze me,
WordPress works out-of-the-box with Apache httpd with only a few lines
of config and Let’s Encrypt works out-of-the-box with Nginx.
However, using them together has proven to be more difficult than I thought
so here’s the nix config I used to create a WordPress server on NixOS:

```nix
{ config, pkgs, ... }:

let

hostName = "0x9b";
FQDN = "${hostName}.moe";

custom-vim = with import <nixpkgs> {}; vim_configurable.customize {
  name = "vim";
  vimrcConfig.customRC = ''
    set nu
    set backspace=indent,eol,start
  '';
  vimrcConfig.vam.pluginDictionaries = [
    { name = "vim-nix"; }
  ];
};

wordpress-generate = type: namee: version: sha256s:
  pkgs.stdenv.mkDerivation rec {
    name = "${namee}-wordpress-${type}";
    src = pkgs.fetchsvn rec {
      url = if (type == "plugin") then
          "http://plugins.svn.wordpress.org/${namee}/${version}"
        else if (type == "theme") then 
          "http://themes.svn.wordpress.org/${namee}/${version}"
        else
          "";
      sha256 = sha256s;
    };
    installPhase = "mkdir -p $out; cp -R * $out/";
  };

# WordPress themes and plugins ===========================

lightpress =
  wordpress-generate
    "theme"
    "lightpress"
    "1.4"
    "05phil64r4cfrcms1ly292910jms9256rh8238qis4s0lq613ya8";
Jetpack =
  wordpress-generate
    "plugin"
    "jetpack"
    "tags/4.6"
    "08dk1b5x2pr5gf2sqza41n0z8liiicalki0fxzkxsblgm81d5qbd";

in

{
  imports = [
    ./hardware-configuration.nix
  ];

  boot.loader.grub.enable = true;
  boot.loader.grub.version = 2;
  boot.loader.grub.device = "/dev/vda";

  networking.hostName = hostName;
  networking.firewall = {
    allowedTCPPorts = [ 80 443 ];
  };

  time.timeZone = "Asia/Hong_Kong";

  environment.systemPackages = with pkgs; [
    wget htop custom-vim
  ];

  services.openssh.enable = true;

  services.mysql = {
    enable = true;
    package = pkgs.mysql;
  };

  services.nginx = {
    enable = true;
    recommendedTlsSettings = true;
    recommendedOptimisation = true;
    recommendedGzipSettings = true;
    recommendedProxySettings = true;
    virtualHosts."${FQDN}" = {
      enableACME = true;
      enableSSL = true;
      forceSSL = true;
      locations."/" = {
        proxyPass = "http://localhost:8000";
      };
    };
  };

  services.httpd = {
    enable = true;
    port = 8000;
    adminAddr = "admin";
    extraSubservices = [
    {
      serviceType = "wordpress";
      themes = [
        lightpress
      ];
      plugins = [
        Jetpack
      ];
      extraConfig = ''
        define('WP_HOME', 'https://${FQDN}');
      define('WP_SITEURL', 'https://${FQDN}');
      $_SERVER['HTTP_HOST'] = ${FQDN};
      $_SERVER['SERVER_PORT'] = 443;
      $_SERVER['HTTPS'] = 'on';
      '';
    }
    ];
  };

  users.extraUsers.akiroz = {
    isNormalUser = true;
    extraGroups = [ "wheel" ];
  };

  # The NixOS release to be compatible with for stateful data such as databases.
  system.stateVersion = "16.09";

}
```

## Refs:
https://nixos.org/wiki/Wordpress
https://cmanios.wordpress.com/2014/04/12/nginx-https-reverse-proxy-to-wordpress-with-apache-http-and-different-port/
