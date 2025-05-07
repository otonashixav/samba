# Samba via Alpine Docker Image

## Usage

This image does not have an entrypoint. Instead, `smbd -F` should be run within `command`. A sample compose.yml: 

```yml
services:
  samba:
    container_name: samba
    image: ghcr.io/otonashixav/samba:main
    restart: always
    ports:
      - 445:445
    configs:
      - source: smbconf
        target: /etc/samba/smb.conf
      - source: smbpasswd
        target: /smbpasswd
        mode: 0600
    volumes:
      - /rootpool/user:/shares:rslave
      - samba_data:/var/lib/samba
    command: >
      /bin/sh -c "
      adduser -HD -u 1100 -s /sbin/nologin -h /dev/null alice;
      adduser -HD -u 1101 -s /sbin/nologin -h /dev/null bob;
      adduser -HD -u 1200 -s /sbin/nologin -h /dev/null family;
      addgroup alice family;
      addgroup bob family;
      smbd -F --no-process-group"
volumes:
  samba_data:

# https://www.samba.org/samba/docs/4.17/man-html/smbpasswd.5.html
configs:
  smbpasswd:
    content: |
      alice:1100:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX:31D6CFE0D16AE931B73C59D7E0C089C0:[U          ]:LCT-681777DB:
      bob:1101:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX:31D6CFE0D16AE931B73C59D7E0C089C0:[U          ]:LCT-681777DB:
  smbconf:
    content: |
      [global]
        client protection = sign
        disable netbios = yes
        disable spoolss = yes
        hosts allow = 192.168.0.0/255.255.0.0 172.16.0.0/255.240.0.0
        load printers = no
        mangled names = no
        passdb backend = smbpasswd:/smbpasswd
        security = user
        server role = standalone
        server string = Samba Server
        smb1 unix extensions = no
        smb ports = 445
        socket options = IPTOS_LOWDELAY TCP_NODELAY
        vfs objects = catia fruit shadow_copy2 streams_xattr
        workgroup = WORKGROUP

        ;log level = 3
        
        catia:mappings = 0x22:0xa8,0x2a:0xa4,0x2f:0xf8,0x3a:0xf7,0x3c:0xab,0x3e:0xbb,0x3f:0xbf,0x5c:0xff,0x7c:0xa6

        fruit:aapl = yes
        fruit:nfs_aces = no
        fruit:resource = stream
        fruit:metadata = stream
        fruit:encoding = native
        fruit:time machine max size = 1T
        fruit:wipe_intentionally_left_blank_rfork = yes
        fruit:delete_empty_adfiles = yes
        
        shadow:snapdir = .zfs/snapshot
        shadow:format = -%Y-%m-%d-%H%M
        shadow:snapprefix = .*
        shadow:delimiter = -20
        
        access based share enum = yes
        acl allow execute always = yes
        acl map full control = no
        create mask = 0640
        directory mask = 0750
        force create mode = 0640
        force directory mode = 0750
        public = no
        writeable = yes
      
      [Alice]
        valid users = alice
        path = /shares/alice
      
      [Bob]
        valid users = bob
        path = /shares/bob
        fruit:time machine = yes
      
      [Family]
        valid users = @family
        path = /shares/family
        force user = family
```
