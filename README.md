# Create a simulator object
set ns [new Simulator]

# Define different colors for data flows (for NAM)
$ns color 1 Blue
$ns color 2 Red

# open the trace file
set tracefile1 [open out.tr w]
$ns trace-all $tracefile1

# Open the NAM trace file
set namfile [open out.nam w]
$ns namtrace-all $namfile

# Define a 'finish' procedure
proc finish {} {
	global ns tracefile1 namfile
	$ns flush-trace
	close $tracefile1
	close $namfile
	exec nam out.nam &
	exit 0
}

# Create 6 nodes
set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]
set n4 [$ns node]
set n5 [$ns node]

# Create links between nodes
$ns duplex-link $n0 $n2 0.5Mb 100ms DropTail
$ns duplex-link $n1 $n2 0.5Mb 100ms DropTail
$ns duplex-link $n2 $n3 0.5Mb 100ms DropTail
$ns duplex-link $n3 $n4 0.5Mb 100ms DropTail
$ns duplex-link $n3 $n5 0.5Mb 100ms DropTail

# Set Queue Sizes
$ns queue-limit $n2 $n3 10
$ns queue-limit $n3 $n4 10
$ns queue-limit $n3 $n5 10

# Setting node positions
$ns duplex-link-op $n0 $n2 orient right-down
$ns duplex-link-op $n1 $n2 orient right-up
$ns duplex-link-op $n2 $n3 orient right
$ns duplex-link-op $n3 $n4 orient right-up
$ns duplex-link-op $n3 $n5 orient right-down

# Defining a Random Uniform Generator
proc Random_Generator {} {
	set MyRng4 [new RNG]
	$MyRng4 seed 0
	set r4 [new RandomVariable/Uniform]
	$r4 use-rng $MyRng4
	$r4 set min_ 0
	$r4 set max_ 4
	puts stdout "Testing Uniform Random Variable inside function"
	global x
	set x [$r4 value]
	return x
}

# Setup a TCP connection
set tcp [new Agent/TCP]
$ns attach-agent $n0 $tcp
set sink [new Agent/TCPSink]
Random_Generator
if {$x < 2.0} {
	$ns attach-agent $n4 $sink
	$ns connect $tcp $sink
} else {
	$ns attach-agent $n5 $sink
	$ns connect $tcp $sink
}

$tcp set fid_ 1
#$tcp set class_ 2
$tcp set packetsize_ 552

#Setup a FTP over TCP connection
set ftp [new Application/FTP]
$ftp attach-agent $tcp
#$ftp set type_ FTP

# Setup a UDP connection
set udp [new Agent/UDP]
$ns attach-agent $n1 $udp
set null [new Agent/Null]
$ns attach-agent $n5 $null
$ns connect $udp $null
$udp set fid_ 2

set cbr [new Application/Traffic/CBR]
$cbr attach-agent $udp
#$cbr set type_ CBR
$cbr set packet_size_ 1000
#$cbr set rate_ 0.01Mb
$cbr set random_ false

#Schedule events for the CBR and FTP agents
$ns at 0.1 "$cbr start"
$ns at 1.0 "$ftp start"
$ns at 4.0 "$ftp stop"
$ns at 4.5 "$cbr stop"

#Call the finish procedure after 5 seconds of simulation time
$ns at 5.0 "finish"



#Run the simulation
$ns run
