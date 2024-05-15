### SampleProfileProbePass
1. BB(except EH or unreachable) and non-intrinsic call are assigned an unique ProbeID.
2. Compute hash for CFG of each function using ProbeID.
3. For each BB(except EH or unreachable), insert a llvm.pseudoprobe intrinsic. Args of llvm.pseudoprobe is (function_guid, BBProbeID, 0, DistributionFactor). e.g.`call void @llvm.pseudoprobe(i64 6699318081062747564, i64 3, i32 0, i64 -1)`
4. For non-intrinsic call, (ProbeID, CallType, 0, DistributionFactor) are packed as upper 32bit of callinst's discriminator.
5. (GUID, FuncHash, FuncName) is stored into !llvm.pseudo_probe_desc metadata.
