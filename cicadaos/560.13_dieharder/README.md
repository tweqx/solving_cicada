# `560.13` cannot be discriminated as non-random by the Dieharder test suite
"[Dieharder](http://webhome.phy.duke.edu/~rgb/General/dieharder.php)" is a popular RNG test suite. It was noticed that running it on `560.13` results in several test failures, as well as many weak tests:
```
$ dieharder -a -g 201 -f 560.13
...
 marsaglia_tsang_gcd    0   10000000      100 0.00000000   FAILED  
 marsaglia_tsang_gcd    0   10000000      100 0.00000000   FAILED  
       diehard_craps    0     200000      100 0.00036928    WEAK   
...
     dab_bytedistrib    0   51200000        1 0.00000000   FAILED  
```
There are 6 "weak" results.

First of all, we know that the first 64167 bytes of `560.13` aren't random as they can be used in an XOR operation to produce [ASCII text](https://pastebin.com/bwvHdXkv). Read the [wiki](https://uncovering-cicada.fandom.com/wiki/User:Ctvrty/sandbox/2013#A_shortcut) for more information. Removing the first 64KB slightly improves the tests.

After many investigations, it was noticed that a random file sampled from `/dev/random` of the same size as `560.13` without the first 64KB fails in a similar way. Thus, it was hypothesized that the file was too short. Dieharder needs a lot of data. Contrary to normal statistical tests, it does not adapt by itself the number of samples drawn to the number of available independant samples. If more samples are needed than that's provided, the input will be rewound. For `560.13`, about 100 (for the GCD tests) and 1000 (for the byte distribution test) rewinds were necessary.

Statistical tests implemented in Dieharder expect samples to be independent, which would explain the differences observed.

Two tests were conducted to check this hypothesis:
- the number of samples was limited such that no rewind was needed. The test did not fail anymore:
  ```
   marsaglia_tsang_gcd|   0|    141374|     100|0.62610476|  PASSED  
   marsaglia_tsang_gcd|   0|    141374|     100|0.68304159|  PASSED  
       dab_bytedistrib|   0|   2283743|       1|0.70194180|  PASSED  
  ```
- 100 files were generated from `/dev/random/` of the same length as `560.13` (minus the 64KB). Dieharder was ran, for both test kinds and the p-values were collected.

  If the tests (without limiting the number of samples) were working correctly, the p-values should be uniformly distributed. An Anderson-Darling test was used to check for this. For both failing tests, the p-value obtained was 0%. This invalidates the null-hypothesis and thus proves that Dieharder with default parameters is not reliable on such small files.

## Contact & Links
`iiisak`, `Mae3301#6221`, `tweqx`

Investigations happened on the CicadaSolvers Discord server. Links:
- [Initial messages](https://discord.com/channels/572330844056715284/615334603049271322/1130379023172780032)
- [Dedicaced thread](https://discord.com/channels/572330844056715284/1130463530123022409)
- [Other messages related](https://discord.com/channels/572330844056715284/787683923383156796/1131707694231867532)
