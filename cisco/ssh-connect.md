# SSH verbinden met Cisco Catalyst 3560

IOS 12.2 gebruikt verouderde algoritmen. Gebruik deze commando vanaf Windows:
`ash
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 \
    -oHostKeyAlgorithms=+ssh-rsa \
    -oCiphers=+aes256-cbc \
    -oMACs=+hmac-sha1 \
    admin@192.168.1.2
`

## Switch IP
192.168.1.2

## Consolekabel
COM4 — 9600 baud (PuTTY)
