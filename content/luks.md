---
title: "LUKS"
date: 2023-04-06T22:54:01+02:00
---

## Backup Header

```bash
cryptsetup luksHeaderBackup <luks device> --header-backup-file <header backup file>
```

## Dump Header

```bash
cryptsetup luksDump <luks device/header backup file>
```

## Test Passphrase

```bash
cryptsetup luksOpen --test-passphrase <luks device/header backup file>
```

## Extract Master Key

```bash
cryptsetup luksDump --dump-master-key <luks device/header backup file>
```

Save it to a file:
```bash
 echo "masterkey" | xxd -r -p > <master key file>
```

## Recover with Master Key

Open the LUKS device:
```bash
cryptsetup luksOpen --master-key-file=<master key file> <luks device> <decrypted block device>
```

Add a new passphrase:
```bash
cryptsetup luksAddKey --master-key-file=<master key file> <luks device>
```