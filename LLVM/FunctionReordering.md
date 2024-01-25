    clang -O2 foo.cpp -ffunction-sections -fsplit-machine-functions -fprofile-sample-use=prof.txt

1. CGProfilePass goes through Module to get \[Caller, Callee, Count\] and save it as "CG Profile" Metadata.
2. TargetLoweringObjectFile::emitCGProfileMetadata convert \[Caller, Callee, Count\] to \[Caller-MCSymbol, Callee-MCSymbol, Count\].
3. MCELFStreamer/MCWinCOFFStreamer stores it into ".llvm.call-graph-profile" section.
4. lld reads this section if it exists.

  COFF specific:
  
5. For each section, lld reads all objects and ".llvm.call-graph-profile".
6. lld uses CallGraphSort to get function priority. It tries to reduce call distance and then sort the function section by density.
   
   
