    clang -O2 foo.cpp -ffunction-sections -fsplit-machine-functions -fprofile-sample-use=prof.txt

1. CGProfilePass goes through Module to get \[Caller, Callee, Count\] and save it as "CG Profile" Metadata.
2. TargetLoweringObjectFile::emitCGProfileMetadata convert \[Caller, Callee, Count\] to \[Caller-MCSymbol, Callee-MCSymbol, Count\].
3. MCELFStreamer/MCWinCOFFStreamer stores it into ".llvm.call-graph-profile" section.
4. lld reads this section for all objects if it exists.

  COFF specific:
  
5. readCallGraphsFromObjectFiles reads all \[Caller-MCSymbol, Callee-MCSymbol, Count\], then converts it to \[Caller-SectionChunk, Callee-SectionChunk, Count\] and store into callGraphProfile.
6. For each partialSections(e.g. .text, .text$hot), lld uses CallGraphSort to get function section priority. It tries to reduce call distance and then sort the function section by density.
   
   
