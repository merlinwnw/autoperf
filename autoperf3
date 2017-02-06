#!/usr/bin/python
import time
import sys
import warnings
import subprocess
import paramiko


port = 22
gate_2 = '192.168.103.112'
gate_1 = '192.168.101.111'
LocTrafGenMachineIP = '192.168.103.28'
RemTrafGenMachineIP = '192.168.101.26'
username = 'root'
password = 'russia'
LocalFileVmstatS1 = '/root/vmstat_s1'
RemoteFileVmstatS1 = '/root/vmstat_s1'
LocalFileVmstatS2 = '/root/vmstat_s2'
RemoteFileVmstatS2 = '/root/vmstat_s2'
StartMbpsForNuttcp = 1000
udp_start_gen = 100
UDP1400 = 1400
UDP512 = 512
UDP64 = 64
esp_c = 'esp_c'
esp_ci = 'esp_ci'
esp_imit = 'esp_imit'
lsp_pass = 'pass'
srv = 'srv'

def StartVmstat (Hostname, Port, Username, Password, RemoteFile):
    vmstat_grep = []
    vmstat_pids = []
    start_vmstat = "vmstat 1 -n > " + str(RemoteFile) + " &"
    paramiko.util.log_to_file('paramiko.log')
    s = paramiko.SSHClient()
    s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    s.connect(Hostname, Port, Username, Password)
    stdin, stdout, stderr = s.exec_command('ps -A | grep vmstat')
    vmstat_grep = stdout.readlines()
    for line in vmstat_grep:
        vmstat_pids.append(line.split()[0])
    if len(vmstat_pids) > 0:
        for i in vmstat_pids:
            stdin, stdout, stderr = s.exec_command("kill " + str(i))
        stdin, stdout, stderr = s.exec_command(start_vmstat)
        time.sleep(3)
        stdin, stdout, stderr = s.exec_command('ps -A | grep vmstat')
        vmstat_current_pid = stdout.read().split()[0]
    else:
        stdin, stdout, stderr = s.exec_command(start_vmstat)
        time.sleep(3)
        stdin, stdout, stderr = s.exec_command('ps -A | grep vmstat')
        vmstat_current_pid = stdout.read().split()[0]
    s.close()
    return vmstat_current_pid

def StopVmstat (Hostname, Port, Username, Password):
    vmstat_grep = []
    vmstat_pids = []
    paramiko.util.log_to_file('paramiko.log')
    s = paramiko.SSHClient()
    s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    s.connect(Hostname, Port, Username, Password)
    stdin, stdout, stderr = s.exec_command('ps -A | grep vmstat')
    vmstat_grep = stdout.readlines()
    for line in vmstat_grep:
        vmstat_pids.append(line.split()[0])
    if len(vmstat_pids) > 0:
        for i in vmstat_pids:
            stdin, stdout, stderr = s.exec_command("kill " + str(i))
    s.close()
def GetVmstatFiles (Hostname, Port, Username, Password, LocalFile, RemoteFile):
    s = paramiko.SSHClient()
    s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    s.connect(Hostname, Port, Username, Password)
    ftp = s.open_sftp()
    ftp.get(LocalFile, RemoteFile)
    s.close()
def ParsingVmstatOutput (LocalFile):
    vmstat_out =[]
    cpu_load = []
    true_cpu_load =[]
    # open file with Vmstat output
    file = open(LocalFile, 'r')
    vmstat_out = file.readlines()
    #head not read
    for line in vmstat_out[3:-1:]:
        #parsing CPU (numbers) load and add their to list
        cpu_load.append(float(line.split()[13]))

    #delete zeros and ones
    if cpu_load.count(0.0) != 0:
        count_zeros = cpu_load.count(0.0)
        for i in range(count_zeros):
            cpu_load.remove(0.0)

    if cpu_load.count(1.0) != 0:
        count_ones = cpu_load.count(1.0)
        for i in range(count_ones):
            cpu_load.remove(1.0)

    #find approximate average of CPU load
    avg = sum(cpu_load)/float(len(cpu_load))

    #create new list where items > approximate average
    for i in cpu_load:
        if i > avg:
            true_cpu_load.append(i)
    return sum(true_cpu_load)/float(len(true_cpu_load))


  
def LSPSwitch(lsp, gate_ip, port, user, passwd):
    if lsp == 'esp_c':
        paramiko.util.log_to_file('paramiko.log')
        s = paramiko.SSHClient()
        s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        s.connect(gate_ip, port, user, passwd)
        stdin, stdout, stderr = s.exec_command('lsp_mgr load -f /opt/lsp/client.311.cp.ESP_C.lsp')
    elif lsp == 'esp_imit':
        paramiko.util.log_to_file('paramiko.log')
        s = paramiko.SSHClient()
        s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        s.connect(gate_ip, port, user, passwd)
        stdin, stdout, stderr = s.exec_command('lsp_mgr load -f /opt/lsp/client.311.cp.ESP_IMIT.lsp')
    elif lsp == 'esp_ci':
        paramiko.util.log_to_file('paramiko.log')
        s = paramiko.SSHClient()
        s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        s.connect(gate_ip, port, user, passwd)
        stdin, stdout, stderr = s.exec_command('lsp_mgr load -f /opt/lsp/client.311.cp.ESP_CI.lsp')
    elif lsp == 'srv':
        paramiko.util.log_to_file('paramiko.log')
        s = paramiko.SSHClient()
        s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        s.connect(gate_ip, port, user, passwd)
        stdin, stdout, stderr = s.exec_command('lsp_mgr load -f /opt/lsp/server.311.cp.lsp')
    elif lsp == 'pass':
        paramiko.util.log_to_file('paramiko.log')
        s = paramiko.SSHClient()
        s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        s.connect(gate_ip, port, user, passwd)
        stdin, stdout, stderr = s.exec_command('lsp_mgr load -f /opt/lsp/pass.lsp')
    ping = subprocess.Popen("ping " + str(RemTrafGenMachineIP) + ' -c 5' , shell=True, executable='/bin/bash', stdout=subprocess.PIPE)

def IperfTCP_test (remote_host_ip):
    vmstat_current_pid_on_s1 = StartVmstat(gate_2, port, username, password, RemoteFileVmstatS1)
    vmstat_current_pid_on_s2 = StartVmstat(gate_1, port, username, password, RemoteFileVmstatS2)
    res = []
    paramiko.util.log_to_file('paramiko.log')
    s = paramiko.SSHClient()
    s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    s.connect(remote_host_ip, port, username, password)
    for i in range(5):
        stdin, stdout, stderr = s.exec_command('killall iperf')
        time.sleep(1)
        stdin, stdout, stderr = s.exec_command('iperf -s')
        time.sleep(1)
        iperf = subprocess.Popen("iperf -t 60" + " -c " + str(remote_host_ip) + ' | grep 0-6' , shell=True, executable='/bin/bash', stdout=subprocess.PIPE)
        iperf_out = iperf.stdout.readlines()
        spd = iperf_out[-1].split()[-2]
        spd = float(spd)
        spd = round(spd,2)
        res.append(spd)
    print(res)
    stdin, stdout, stderr = s.exec_command('killall iperf')
    GetVmstatFiles(gate_2, port, username, password, LocalFileVmstatS1, RemoteFileVmstatS1)
    GetVmstatFiles(gate_1, port, username, password, LocalFileVmstatS2, RemoteFileVmstatS2)
    cpu_load_on_site1 = ParsingVmstatOutput(LocalFileVmstatS1)
    cpu_load_on_site2 = ParsingVmstatOutput(LocalFileVmstatS2)
    print(round(cpu_load_on_site1), "CPU load on Site_1")
    print(round(cpu_load_on_site2), "CPU load on Site_2")



def IperfTCP_dual_test (remote_host_ip):
    vmstat_current_pid_on_s1 = StartVmstat(gate_2, port, username, password, RemoteFileVmstatS1)
    vmstat_current_pid_on_s2 = StartVmstat(gate_1, port, username, password, RemoteFileVmstatS2)
    res = []
    paramiko.util.log_to_file('paramiko.log')
    s = paramiko.SSHClient()
    s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    s.connect(remote_host_ip, port, username, password)
    for i in range(5):
        stdin, stdout, stderr = s.exec_command('killall iperf')
        time.sleep(1)
        stdin, stdout, stderr = s.exec_command('iperf -s')
        time.sleep(1)
        iperf = subprocess.Popen("iperf -t 60 " + " -c " + str(remote_host_ip) + ' -d | grep 0-6' , shell=True, executable='/bin/bash', stdout=subprocess.PIPE)
        iperf_out = iperf.stdout.readlines()
        spd1 = iperf_out[-1].split()[-2]
        spd2 = iperf_out[-2].split()[-2]
        spd = float(spd1) + float(spd2)
        spd = round(spd,2)
        res.append(spd)
    print(res)
    stdin, stdout, stderr = s.exec_command('killall iperf')
    GetVmstatFiles(gate_2, port, username, password, LocalFileVmstatS1, RemoteFileVmstatS1)
    GetVmstatFiles(gate_1, port, username, password, LocalFileVmstatS2, RemoteFileVmstatS2)
    cpu_load_on_site1 = ParsingVmstatOutput(LocalFileVmstatS1)
    cpu_load_on_site2 = ParsingVmstatOutput(LocalFileVmstatS2)
    print(round(cpu_load_on_site1), "CPU load on Site_1")
    print(round(cpu_load_on_site2), "CPU load on Site_2")


def IperfTCP_dual_p2_test (remote_host_ip):
    vmstat_current_pid_on_s1 = StartVmstat(gate_2, port, username, password, RemoteFileVmstatS1)
    vmstat_current_pid_on_s2 = StartVmstat(gate_1, port, username, password, RemoteFileVmstatS2)
    res = []
    global udp_start_gen
    paramiko.util.log_to_file('paramiko.log')
    s = paramiko.SSHClient()
    s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    s.connect(remote_host_ip, port, username, password)
    for i in range(5):
        stdin, stdout, stderr = s.exec_command('killall iperf')
        time.sleep(1)
        stdin, stdout, stderr = s.exec_command('iperf -s')
        time.sleep(1)
        iperf = subprocess.Popen("iperf -t 60" + " -c " + str(remote_host_ip) + ' -d -P2 | grep 0-6 | grep SUM' , shell=True, executable='/bin/bash', stdout=subprocess.PIPE)
        iperf_out = iperf.stdout.readlines()
        spd1 = iperf_out[-1].split()[-2]
        spd2 = iperf_out[-2].split()[-2]
        spd = float(spd1) + float(spd2)
        spd = float(spd1) + float(spd2)
        spd = round(spd,2)
        res.append(spd)
    print(res)
    stdin, stdout, stderr = s.exec_command('killall iperf')
    GetVmstatFiles(gate_2, port, username, password, LocalFileVmstatS1, RemoteFileVmstatS1)
    GetVmstatFiles(gate_1, port, username, password, LocalFileVmstatS2, RemoteFileVmstatS2)
    cpu_load_on_site1 = ParsingVmstatOutput(LocalFileVmstatS1)
    cpu_load_on_site2 = ParsingVmstatOutput(LocalFileVmstatS2)
    print(round(cpu_load_on_site1), "CPU load on Site_1")
    print(round(cpu_load_on_site2), "CPU load on Site_2")
    udp_start_gen = (sum(res)/5)*1.7

def IperfFullTCP_test():
    print('one way tcp')
    IperfTCP_test (RemTrafGenMachineIP)
    print('tcp dual')
    IperfTCP_dual_test (RemTrafGenMachineIP)
    print('tcp dual p2')
    IperfTCP_dual_p2_test (RemTrafGenMachineIP)

def NuttcpUDP_test_modd (Mbps, PacketLenght, RemoteHostIP):
    _spd = 0
    spd = 0
    if udp_start_gen < 1000:
        gen = udp_start_gen
    else:
        gen = Mbps
    rgen = 0
    loss = 0
    while 1:
        nuttcp = subprocess.Popen("nuttcp -u -T30" + " -Ri" + str(gen) + "m" + " -l" + str(PacketLenght) + " " + str(RemoteHostIP), shell=True, executable='/bin/bash', stdout=subprocess.PIPE)
        nuttcpOutput = nuttcp.stdout.readlines()
        spd = nuttcpOutput[-1].split()[6]
        loss = nuttcpOutput[-1].split()[-2]
        rgen = (float(spd)/( 100 - float(loss))) * 100
        rgen = round(rgen,2)
        print("GEN:", gen, " SPEED:", spd, " LOST:", loss, " REAL_GEN:", rgen)
        if float(loss) > 0:
            print("loss > 0")
#			if ( 100 * (float(rgen) / float(gen))) < 90:
#				print "(float(rgen) / float(gen))) < 0.93"
#				gen = rgen
#			else:
#				print "(float(rgen) / float(gen))) >= 0.93"
            if float(loss) < 0.5:
                print("float(loss) < 0.5")
##			if float(loss) > 0:
                if float(loss) == 0:
                    print('loss = 0')
                    gen = 1.05 * gen
                else:
                    return gen
                    break
            gen = (float(gen) + float(spd))/2
            gen = round(gen,2)
        else:
            gen = 1.05 * gen
            print("loss = 0")
#			return gen
#			break

def NuttcpUDP_test_mod (Mbps, PacketLenght, RemoteHostIP):
    _spd = 0
    spd = 0
    if udp_start_gen < 1000:
        gen = udp_start_gen
    else:
        gen = Mbps
    rgen = 0
    loss = 0
    while 1:
        nuttcp = subprocess.Popen("nuttcp -u -T30" + " -Ri" + str(gen) + "m" + " -l" + str(PacketLenght) + " " + str(RemoteHostIP), shell=True, executable='/bin/bash', stdout=subprocess.PIPE)
        nuttcpOutput = nuttcp.stdout.readlines()
        spd = nuttcpOutput[-1].split()[6]
        loss = nuttcpOutput[-1].split()[-2]
        rgen = (float(spd)/( 100 - float(loss))) * 100
        rgen = round(rgen,2)
        print("GEN:", gen, " SPEED:", spd, " LOST:", loss, " REAL_GEN:", rgen)
        if float(loss) > 0:
#			print "loss > 0"
            if ( 100 * (float(rgen) / float(gen))) < 93:
#				print "(float(rgen) / float(gen))) < 0.93"
                gen = 1.07 * rgen
            else:
#				print "(float(rgen) / float(gen))) >= 0.93"
                if float(loss) < 0.5:
#					print "float(loss) < 0.5"
##			if float(loss) > 0:
                    return gen
                    break
                gen = (float(gen) + float(spd))/2
                gen = round(gen,2)
        else:
#			print "loss = 0"
            return gen
            break

def NuttcpProductionTest (PacketLenght, RemoteHostIP):
    Speed = NuttcpUDP_test_mod (StartMbpsForNuttcp, PacketLenght, RemoteHostIP)
    print(Speed)
    e = 5
    res = []
    vmstat_current_pid_on_s1 = StartVmstat(gate_2, port, username, password, RemoteFileVmstatS1)
    vmstat_current_pid_on_s2 = StartVmstat(gate_1, port, username, password, RemoteFileVmstatS2)
    while e != 0:
        nuttcp = subprocess.Popen("nuttcp -u -T60" + " -Ri " + str(Speed) + "m" + " -l " + str(PacketLenght) + " " + str(RemoteHostIP), shell=True, executable='/bin/bash', stdout=subprocess.PIPE)
        nuttcpOtputs = nuttcp.stdout.readlines()
        speed_wo_loss = nuttcpOtputs[-1].split()[6]
        loss = nuttcpOtputs[-1].split()[-2]
        print(speed_wo_loss, "Mbps, PacketLenght is", PacketLenght, "lost", loss)
        res.append(round(float(speed_wo_loss),2))
        e = e - 1
    GetVmstatFiles(gate_2, port, username, password, LocalFileVmstatS1, RemoteFileVmstatS1)
    GetVmstatFiles(gate_1, port, username, password, LocalFileVmstatS2, RemoteFileVmstatS2)
    cpu_load_on_site1 = ParsingVmstatOutput(LocalFileVmstatS1)
    cpu_load_on_site2 = ParsingVmstatOutput(LocalFileVmstatS2)
    print(res)
    print(round(cpu_load_on_site1), "CPU load on Site_1")
    print(round(cpu_load_on_site2), "CPU load on Site_2")



LSPSwitch(srv, gate_1, port, username, password)
LSPSwitch(esp_c, gate_2, port, username, password)
print(esp_c)
IperfFullTCP_test()
NuttcpProductionTest (UDP1400, RemTrafGenMachineIP)
NuttcpProductionTest (UDP512, RemTrafGenMachineIP)
NuttcpProductionTest (UDP64, RemTrafGenMachineIP)
LSPSwitch(esp_imit, gate_2, port, username, password)
print(esp_imit)
IperfFullTCP_test()
NuttcpProductionTest (UDP1400, RemTrafGenMachineIP)
NuttcpProductionTest (UDP512, RemTrafGenMachineIP)
NuttcpProductionTest (UDP64, RemTrafGenMachineIP)
LSPSwitch(esp_ci, gate_2, port, username, password)
print(esp_ci)
IperfFullTCP_test()
NuttcpProductionTest (UDP1400, RemTrafGenMachineIP)
NuttcpProductionTest (UDP512, RemTrafGenMachineIP)
NuttcpProductionTest (UDP64, RemTrafGenMachineIP)
LSPSwitch(lsp_pass, gate_1, port, username, password)
LSPSwitch(lsp_pass, gate_2, port, username, password)
print(lsp_pass)
IperfFullTCP_test()
NuttcpProductionTest (UDP1400, RemTrafGenMachineIP)
NuttcpProductionTest (UDP512, RemTrafGenMachineIP)
NuttcpProductionTest (UDP64, RemTrafGenMachineIP)

