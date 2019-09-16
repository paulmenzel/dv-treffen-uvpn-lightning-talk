% uvpn – VPN aus dem eigenen Netz in andere Netze
% Paul Menzel (Max-Planck-Institut für molekulare Genetik)
% 19. September 2019

# Probleme

## Anforderung

1.  Beschäftigte, die auch an anderen Einrichtungen arbeiten, oder Gäste brauchen VPN zum Zugriff auf die EDV-Umgebung
1.  Meist OpenVPN-Klient installieren

## Probleme

1.  Umgeht Firewall
1.  Klient läuft mit Adminrechten.
1.  OpenVPN-Server schickt Konfiguration.

# Lösung

1.  Umsetzung von Donald Buczek
1.  Gästenetz und Linux-Kernel-Namespaces
1.  Behandlung wie externes Gerät im Gästenetz
1.  Umsetzung mit Bash-Skript [uvpn](https://github.molgen.mpg.de/mariux64/mxtools/blob/master/uvpn/uvpn)
1.  Zugriff auf Dateien wie normaler Nutzer\*in

# Nutzung

```
$ uvpn/uvpn
  usage:
    uvpn/uvpn start           # start uvpn container
    uvpn/uvpn stop            # stop uvpn container
    uvpn/uvpn exec [cmd...]   # execute cmd (default bash) in the container
    uvpn/uvpn show [user...]  # show processes

    uvpn/uvpn start_as_root    # sudo callback - internal use
    uvpn/uvpn stop_as_root     # sudo callback - internal use
```

# Priviligierte Operation

1.  Erzeugen des Network-Namespaces priviligiert
1.  `/etc/sudoers` (`sudo visudo`)

        ALL ALL=NOPASSWD: /usr/bin/uvpn start_as_root,/usr/bin/uvpn stop_as_root

# Auswahl

Funktion `cmd_start_as_root()`:

```
mkdir -p "/run/uvpn/$user/ns"
chown "$user" "/run/uvpn/$user"
for ns in user net mnt;do
	touch "/run/uvpn/$user/ns/$ns"
done
mount --bind "/run/uvpn/$user/ns" "/run/uvpn/$user/ns"
mount --make-private "/run/uvpn/$user/ns"

pid=$(setuid $uid unshare --mount --user --net bash -c 'sleep 30 > /dev/null&echo $!')
for ns in user net mnt; do
	mount --bind /proc/$pid/ns/$ns "/run/uvpn/$user/ns/$ns"
done
echo "0 $uid 1" > /proc/$pid/uid_map
echo "0 $(id -g "$user") 1" > /proc/$pid/gid_map
ip link set "uvpn.$user.1" netns $pid
kill $pid
```
