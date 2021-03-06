---
title: MediaWiki
permalink: /MediaWiki/
---

Here is the NixOS configuration that you need to get a MediaWiki instance:

### mediawiki but NO vhosts yet

    services.postgresql.enable = true;
    services.postgresql.package = pkgs.postgresql92;
    services.httpd.enable = true;
    services.httpd.adminAddr = "admin@example.org";
    services.httpd.extraSubservices =
      [ { serviceType = "mediawiki";
          siteName = "My Wiki";
          logo = "http://www.example.org/wiki-logo.png"; # should be 135x135px
          extraConfig =
            ''
              # See http://www.mediawiki.org/wiki/Manual:Configuration_settings
              $wgEmailConfirmToEdit = true;
            '';
        }
      ];

Then run `nixos-rebuild switch`. The MediaWiki database will be created automatically.

You should also set up a user account for the administrator. This can be done from the command line:

`
$ mediawiki-main-createAndPromote --bureaucrat God foobar
`

Note that various other MediaWiki commands are also accessible from the command line. For instance:

`
$ mediawiki-main-changePassword --user alice --password foobar
`

See also: /etc/nixos/nixos/modules/services/web-servers/apache-httpd/mediawiki.nix

### mediawiki and vhosts

this example features two vhost entries:

-   a normal webpage with an index.html
-   a mediawiki presence (which is found at wiki.example.com or wiki.example.com/mywiki

<!-- -->

      services.postgresql.enable = true;
      services.postgresql.package = pkg.postgresql92;

      services.httpd = {
        enable = true;
        logPerVirtualHost = true;
        adminAddr="example@example.com";
        hostName = "example.com";

        virtualHosts =
        [
          {
            hostName = "www.example.com";
            serverAliases = ["www.example.com"];
            documentRoot = "/www";
          }
         {
            # Note: do not forget to add a DNS entry for wiki.example.com in the DNS settings
            hostName = "wiki.example.com";
            extraConfig = ''
                RedirectMatch ^/$ /mywiki
              '';
            extraSubservices =
            [
              {
                serviceType = "mediawiki";
                siteName = "My Wiki";
                articleUrlPrefix = "/mywiki";
                #logo = "http://www.example.org/wiki-logo.png"; # should be 135x135px
                extraConfig =
                  ''
                    # See http://www.mediawiki.org/wiki/Manual:Configuration_settings
                    $wgEmailConfirmToEdit = true;
                  '';
              }
            ];
          }
        ];
      };

For virtual hosts: <https://svn.nixos.org/repos/nix/configurations/trunk/tud/cartman.nix>

### 2 or more mediawiki instances using vhosts

notice the added new attributes:

-   id="wiki1"
-   dbName="mediawiki_wiki1"

the second attribute set (remember? { ... } &lt;- this way looks an attribute set) is just a copy of the first with some modifications as the id and dbname must be different, as well as some other values.

      services.postgresql.enable = true;

      services.httpd = {
        enable = true;
        logPerVirtualHost = true;
        adminAddr="example@example.com";
        hostName = "example.com";

        virtualHosts =
        [
          {
            hostName = "www.example.com";
            serverAliases = ["www.example.com"];
            documentRoot = "/www";
          }
         {
            # Note: do not forget to add a DNS entry for wiki.example.com in the DNS settings
            hostName = "wiki.example.com";
            extraConfig = ''
                RedirectMatch ^/$ /mywiki
              '';
            extraSubservices =
            [
              {
                serviceType = "mediawiki";
                id="wiki1";
                dbName="mediawiki_wiki1";

                siteName = "My Wiki";
                articleUrlPrefix = "/mywiki";
                #logo = "http://www.example.org/wiki-logo.png"; # should be 135x135px
                extraConfig =
                  ''
                    # See http://www.mediawiki.org/wiki/Manual:Configuration_settings
                    $wgEmailConfirmToEdit = true;
                  '';
              }
            ];
          }
          {
            # Note: do not forget to add a DNS entry for wiki.example.com in the DNS settings
            hostName = "wiki2.example.com";
            extraConfig = ''
                RedirectMatch ^/$ /mywiki2
              '';
            extraSubservices =
            [
              {
                serviceType = "mediawiki";
                id="wiki2";
                dbName="mediawiki_wiki2";
                siteName = "wiki 2";

                siteName = "My Wiki";
                articleUrlPrefix = "/mywiki";
                #logo = "http://www.example.org/wiki-logo.png"; # should be 135x135px
                extraConfig =
                  ''
                    # See http://www.mediawiki.org/wiki/Manual:Configuration_settings
                    $wgEmailConfirmToEdit = true;
                  '';
              }
            ];
          }
        ];
      };