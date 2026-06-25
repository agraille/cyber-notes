Scan ip : 10.129.68.215


Sous Domaine :


USER:


Tools et versions:


CMD:




















Snapped is a hard-difficulty machine that features two recent CVEs. The foothold showcases CVE-2026-27944 in Nginx-UI, which exposes the /api/backup endpoint without authentication. The endpoint will produce a full backup of the nginx and nginx-UI configuration files, and includes the key to decrypt the backup in the response headers. This leads to finding and decrypting a weak user password from the Nginx-UI database file. Root exploits CVE-2026-3888, a TOCTOU race condition between snap-confine and systemd-tmpfiles. After the system's cleanup daemon deletes a stale mimic directory under /tmp, the attacker recreates it with controlled content and single-steps snap-confine's execution via AF_UNIX socket backpressure to win the race during the mimic bind-mount sequence reliably. This poisons the sandbox's shared libraries, enabling dynamic linker hijacking on the SUID-root snap-confine binary to compromise the system.
