# Synology Directory Server Hash Extractor

## Introduction

Some small businesses may use Synology Directory Server instead of a fully fledged Windows Server for a variety of reasons. Such a configuration is nearly identical in functionality, however, not fully. I came to be in a situation where I needed to test the strength of passwords at a given company, and that company was using their Synology NAS as their Domain Controller. Normally, I would use Mimikatz's DCSync module or manual extraction via sam.hiv and system.hiv to extract the hashes, however, DCSync seems to be broken (or otherwise disabled) with Synoloy's implementation and manual extraction the way that I was used to doing would be rather difficult since Synolgoy is running Linux, not Windows. 

After some searching, I did not find any tools made to extract AD hashes from a Synology NAS running Directory Server, so I made one myself. :)

## Usage

```
usage: extract.py [-h] [--path [directory]] [--output [output file]]

optional arguments:
  -h, --help            show this help message and exit
  --path [directory]    specify location of private ldap data folder (default:
                        /volume*/@appstore/DirectoryServerForWindowsDomain/private/)
  --output [output file]
                        output file name
```

This script can be run with either python2.7 or with python3 on the target Synology NAS or locally. If the script is to be run locally, the following folder needs to be copied from the Synology.

```
/volume*/@appstore/DirectoryServerForWindowsDomain/private
```


## Example Usage

On a Synology NAS running Directory Server

```
admin@synology:/tmp$ sudo python dump.py
-+= CRFSlick's Synology Directory Server Hash Extractor =+-
[*] Running command... Done!
[*] Writing output to "output.dat"
[+] Success!
admin@synology:/tmp$ cat output.dat 
jsmith:af618854faa213f29760a9dd225cafc4
tmike:4b287445eeb20f4f5c4c5f888e970c6b
admin:841a370075da4fa12968d5b58e8ffdaa
(...)
```

On local machine with copy of `/volume*/@appstore/DirectoryServerForWindowsDomain/private`

```
slick㉿kubuntu:~$ python3 extract.py --path /home/slick/private/ --output hashes.dat
-+= CRFSlick's Synology Directory Server Hash Extractor =+-
[*] Running command... Error!
[!] It seems that running "ldbsearch" has failed, resorting to manual extraction.
[*] Writing output to "hashes.dat"
[+] Success!
slick㉿kubuntu:~$ cat hashes.dat 
jsmith:af618854faa213f29760a9dd225cafc4
tmike:4b287445eeb20f4f5c4c5f888e970c6b
admin:841a370075da4fa12968d5b58e8ffdaa
(...)
```


## Dependencies

My Synology with Directory Server installed already had `ldapsearch`, so I assume that it's standard. You only need to have the python dependencies installed if you are running this locally.

+ Programs / Binaries
  + ldapasearch
+ Python Modules
  + samba
  + ldb

## Other Information

I also made this quick, dirty bash one-liner to extract just the hashes. This *will not* show you what user or computer the hashes belong to and relies on having `ldbsearch` installed, but it's nice for quick extraction. Replace `sam.ldb` with your relevant file location.

```sh
data=$(ldbsearch -H sam.ldb unicodepwd | grep 'unicodePwd' | awk -F ' ' {'print $2'}); while IFS= read -r line; do python -c "import codecs ; import binascii; print (binascii.hexlify(codecs.decode( '$line' , 'base64')))"; done <<< $data
```

```
slick㉿kubuntu:~$ data=$(ldbsearch -H sam.ldb unicodepwd | grep 'unicodePwd' | awk -F ' ' {'print $2'}); while IFS= read -r line; do python -c "import codecs ; import binascii; print (binascii.hexlify(codecs.decode( '$line' , 'base64')))"; done <<< $data 
af618854faa213f29760a9dd225cafc4
4b287445eeb20f4f5c4c5f888e970c6b
841a370075da4fa12968d5b58e8ffdaa
(...)
```

## Disclaimer

This project is intended for educational purposes only and cannot be used for law violation or personal gain. The author of this project is not responsible for any possible harm caused by the materials. 
