# Bash

### What does gunzip do?

<details>
`gunzip` is a command-line utility used in Unix and Unix-like operating systems to decompress files compressed by the `gzip` utility. 

The `gzip` (GNU zip) utility is used to compress files, typically to save space or to reduce the time needed to transmit the files over the network. The compressed files usually have the extension `.gz`.

When you want to restore the original file from a `gzip`-compressed file, you use `gunzip`. The basic syntax is:

```bash
gunzip filename.gz
```

This command will decompress the file `filename.gz` and the result is the original file `filename`. The compressed file `filename.gz` is removed.

If you want to keep the original `.gz` file after decompression, you can use the `-k` or `--keep` option:

```bash
gunzip -k filename.gz
```

This command will create the decompressed file `filename` in the same directory and keep the original `filename.gz` file.

</details>



###
###
###
###

### What is the difference between wget and curl?

### What options are avaliable for wget? 