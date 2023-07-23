# 560.13 is not an OpenPGP Secret Key
`file` reports `560.13` has being an "OpenPGP Secret Key". This is a false positive.
```
$ file 560.13
560.13: OpenPGP Secret Key
```

More specifically, `file` identifies it has a binary OpenPGP secret file. `file` uses magic files, a special scripting language to describe how a type format should be parsed. The magic file for the binary secret key format can be found [there](https://github.com/file/file/blob/269b5fc0fb81631e41458fe3a63254d5593335e4/magic/Magdir/pgp-binary-keys).

Lines starting with `#` are comments. Most other lines are instructions of how to parse the file format. The file is read sequentially. Each line corresponds to a test to perform, or to a subroutine to call. `>` characters indicate the indentation level. See the `magic.5` man page for more information.

The first instructions are:
```
0	ubyte			=0xC6	OpenPGP Public Key
>&0	use			primary_key_length_new
# New-Style Secret Key
0	ubyte			=0xC5	OpenPGP Secret Key
>&0	use			primary_key_length_new
# Old-Style Public Key
0	ubyte&0xFC		=0x98	OpenPGP Public Key
>&-1	use			primary_key_length_old
# Old-Style Secret Key
0	ubyte&0xFC		=0x94	OpenPGP Secret Key
>&-1	use			primary_key_length_old
```
meaning that is the first byte equals to `0xC6`, print `OpenPGP Public Key` and call the `primary_key_length_new` subroutine. Otherwise, the call to the subroutine will be skipped and the next check will be performed. 

For `560.13`, the fourth check suceeds, resulting in "OpenPGP Secret Key" being printed. The 6 most significant bits of the first byte were compared to a given value. As soon as at least a message is printed, `file` stops checking other file format. Consequently, random data files will have 4/256 = 1.5625 % of resulting in "OpenPGP Secret Key" being printed.

The next bytes don't make sense for a binary OpenPGP secret key however, as confirmed by running `pgpdump`:
```
$ pgpdump 560.13
Old: Secret Key Packet(tag 5)(-1900171438 bytes)
pgpdump: unknown version (103).
	Ver 103 - 
```

This is also why `file` does not print anything else, contrary to what it would do for a legitimate binary key:
```
$ gpg --export-secret-key > /tmp/secret_key ; file /tmp/secret_key ; rm /tmp/secret_key
/tmp/secret_key: OpenPGP Secret Key Version 4, Created XXX XXX XX XX:XX:XX XXXX, RSA (Encrypt or Sign, 4096 bits); User ID; Signature; OpenPGP Certificate
```

Actually, it's possible to replicate the same output just by using a file made of 2 bytes (as for only 1 byte, `file` reports it a `very short file (no magic)`):
```
$ echo -ne "\x96\x8e" > /tmp/experiment ; file /tmp/experiment
/tmp/experiment: OpenPGP Secret Key
```

The binary pgp key format was added to `file` in 2020 ([git blame](https://github.com/file/file/blob/269b5fc0fb81631e41458fe3a63254d5593335e4/magic/Magdir/pgp-binary-keys)), meaning that it's unlikely that 3301 purposely chose the first byte to result in a false positive.

## Contact
`tweqx`
