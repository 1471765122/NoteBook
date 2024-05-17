### SampleProfileProbePass
Insert llvm.pseudoprobe and update calllinst's discriminator(DWARF).
1. BB(except EH or unreachable) and non-intrinsic call are assigned an unique ProbeID.
2. Compute hash for CFG of each function using ProbeID.
3. For each BB(except EH or unreachable), insert a llvm.pseudoprobe intrinsic. Args of llvm.pseudoprobe is `(GUID, BBProbeID, Attr, DistributionFactor)`. e.g.`call void @llvm.pseudoprobe(i64 6699318081062747564, i64 3, i32 0, i64 -1)`
4. For non-intrinsic call, `(ProbeID, CallType, 0, DistributionFactor)` are packed as upper 32bit of callinst's discriminator. This helps build ProbeID CallStack.
5. `(GUID, FuncHash, FuncName)` is stored into !llvm.pseudo_probe_desc metadata.

### PseudoProbeUpdatePass
Update DistributionFactor of llvm.pseudoprobe and callinst.
e.g. Given a BB with ProbeID `N`. If it is copied due to inlining or something else. There're multiple BB with same ProbeID `N`. Here PseudoProbeUpdatePass will update DistributionFactor:
1. Aggregate all BlockProfileCount of BB with same ProbeID and CallStack hash as `sum`.
2. DistributionFactor = BBProfileCount(BB) / `sum`.

### PseudoProbeVerifier
Verifiy whether DistributionFactor has changed.

### PseudoProbeInserter
1. Convert callinst's discriminator to TargetOpcode::PSEUDO_PROBE MachineInstr. GUID of PSEUDO_PROBE is set to callsite function's GUID.
2. Reoder PSEUDO_PROBE before any real instruction.
3. Remove dangling PSEUDO_PROBE.

## `-fpseudo-probe-for-profiling`
1. Insert SampleProfileProbePass at the pipeline begin. It inserts pseudoprobe intrinsic and update callinst's discriminator.
2. SelectionDAG lower pseudoprobe intrinsic to ISD::PSEUDO_PROBE. Then emit TargetOpcode::PSEUDO_PROBE MachineInstr. Only (GUID, ProbeID, BBType, Attr) is kept.
3. Run PseudoProbeInserter in CG. 
4. PseudoProbeHandler build InlineStack via discriminator and call emitPseudoProbe in MCStreamer.

   
2. Insert PseudoProbeUpdatePass at the end of IR pass pipeline when specified `-fprofile-sample-use`.


