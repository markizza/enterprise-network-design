# Device Configurations

One file per device, named for its hostname: `R1-HQ.txt`, `R2-REMOTE.txt`, `SW-DISTRIBUTION.txt`, `SW-ACCESS-1.txt`, `SW-ACCESS-2.txt`, `SW-SERVER.txt`.

**Before I commit anything here** I replace the enable secret hashes, local account passwords, the OSPF MD5 key and the IPsec pre-shared key with `<redacted>`. They're lab values on a simulated network and carry no real risk — but I'd rather build the habit on something harmless than discover I don't have it on something that isn't.
