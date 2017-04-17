# G1GC检查

早期JDK 8 HotSpotJVM版本在开启G1GC回收器时会存在已知的问题会导致索引腐化，JDK 8u40之前的版本都会受影响。G1GC检查探测是否是之前的HotSpot JVM版本。