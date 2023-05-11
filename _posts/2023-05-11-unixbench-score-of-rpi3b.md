UnixBench score for Raspberry Pi 3B+
-------------

                
				

    
       #    #  #    #  #  #    #          #####   ######  #    #   ####   #    #
       #    #  ##   #  #   #  #           #    #  #       ##   #  #    #  #    #
       #    #  # #  #  #    ##            #####   #####   # #  #  #       ######
       #    #  #  # #  #    ##            #    #  #       #  # #  #       #    #
       #    #  #   ##  #   #  #           #    #  #       #   ##  #    #  #    #
        ####   #    #  #  #    #          #####   ######  #    #   ####   #    #
    
       Version 5.1.3                      Based on the Byte Magazine Unix Benchmark
    
       Multi-CPU version                  Version 5 revisions by Ian Smith,
                                          Sunnyvale, CA, USA
       January 13, 2011                   johantheghost at yahoo period com
    
    
    1 x Dhrystone 2 using register variables  1 2 3 4 5 6 7 8 9 10
    
    1 x Double-Precision Whetstone  1 2 3 4 5 6 7 8 9 10
    
    1 x Execl Throughput  1 2 3
    
    1 x File Copy 1024 bufsize 2000 maxblocks  1 2 3
    
    1 x File Copy 256 bufsize 500 maxblocks  1 2 3
    
    1 x File Copy 4096 bufsize 8000 maxblocks  1 2 3
    
    1 x Pipe Throughput  1 2 3 4 5 6 7 8 9 10
    
    1 x Pipe-based Context Switching  1 2 3 4 5 6 7 8 9 10
    
    1 x Process Creation  1 2 3
    
    1 x System Call Overhead  1 2 3 4 5 6 7 8 9 10
    
    1 x Shell Scripts (1 concurrent)  1 2 3
    
    1 x Shell Scripts (8 concurrent)  1 2 3
    
    ========================================================================
       BYTE UNIX Benchmarks (Version 5.1.3)
    
       System: : GNU/Linux
       OS: GNU/Linux -- 5.10.161 -- #0 SMP Tue Jan 3 00:24:21 2023
       Machine: aarch64 (unknown)
       Language:  (charmap=, collate=)
       02:01:48 up  1:37,  load average: 0.00, 0.00, 0.00; runlevel 
    
    ------------------------------------------------------------------------
    Benchmark Run: Tue Jan 03 2023 02:01:48 - 02:29:50
    0 CPUs in system; running 1 parallel copy of tests
    
    Dhrystone 2 using register variables        7814526.3 lps   (10.0 s, 7 samples)
    Double-Precision Whetstone                     1928.6 MWIPS (10.0 s, 7 samples)
    Execl Throughput                               1108.1 lps   (29.9 s, 2 samples)
    File Copy 1024 bufsize 2000 maxblocks        162733.4 KBps  (30.0 s, 2 samples)
    File Copy 256 bufsize 500 maxblocks           48420.9 KBps  (30.0 s, 2 samples)
    File Copy 4096 bufsize 8000 maxblocks        393693.5 KBps  (30.0 s, 2 samples)
    Pipe Throughput                              367779.2 lps   (10.0 s, 7 samples)
    Pipe-based Context Switching                  59328.5 lps   (10.0 s, 7 samples)
    Process Creation                               2353.3 lps   (30.0 s, 2 samples)
    Shell Scripts (1 concurrent)                   1854.2 lpm   (60.0 s, 2 samples)
    Shell Scripts (8 concurrent)                    604.1 lpm   (60.0 s, 2 samples)
    System Call Overhead                         484856.2 lps   (10.0 s, 7 samples)
    
    System Benchmarks Index Values               BASELINE       RESULT    INDEX
    Dhrystone 2 using register variables         116700.0    7814526.3    669.6
    Double-Precision Whetstone                       55.0       1928.6    350.7
    Execl Throughput                                 43.0       1108.1    257.7
    File Copy 1024 bufsize 2000 maxblocks          3960.0     162733.4    410.9
    File Copy 256 bufsize 500 maxblocks            1655.0      48420.9    292.6
    File Copy 4096 bufsize 8000 maxblocks          5800.0     393693.5    678.8
    Pipe Throughput                               12440.0     367779.2    295.6
    Pipe-based Context Switching                   4000.0      59328.5    148.3
    Process Creation                                126.0       2353.3    186.8
    Shell Scripts (1 concurrent)                     42.4       1854.2    437.3
    Shell Scripts (8 concurrent)                      6.0        604.1   1006.8
    System Call Overhead                          15000.0     484856.2    323.2
                                                                       ========
    System Benchmarks Index Score                                         365.9
    
    




**Hardware**: BCM2835
Model: Raspberry Pi 3 Model B Plus Rev 1.3

**Software**: OpenWrt 22.03 64bit
UnixBench 5.1.3

For unknown reason it detects 0 CPU therefor only gives score for single core. Any hints please let me know via Twitter, thanks.


