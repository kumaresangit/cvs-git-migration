Setting Up CVS Server on CentOS 7
  This document provides step-by-step instructions to install and configure a CVS server on CentOS 7.

Step 1: Install Required Packages
  1.	Install CVS and xinetd:
      yum install cvs xinetd
  2.	Verify CVS Installation:
      cvs -v

Step 2: Configure Firewall
  1.	Add Firewall Rules:
      firewall-cmd --permanent --add-port=2401/tcp
      firewall-cmd --permanent --add-port=2401/udp
  2.	Reload Firewall:
      firewall-cmd --reload
  3.	Verify Firewall Rules:
      firewall-cmd --list-all

Step 3: Create CVS User and Initialize Repository
  1.	Create CVS User:
      useradd cvs
  2.	Set Password for CVS User:
      passwd cvs
      # (Enter the password "cvs" when prompted)
  3.	Create Repositories Directory:
      mkdir /home/cvs/Repositories
  4.	Initialize CVS Repository:
      cvs -d /home/cvs/Repositories init

Step 4: Restore Repository from Backup
  1.	Navigate to CVS Home Directory:
      cd /home/cvs
  2.	Extract Repository Backup:
      tar -xvzf Repositories_20_05_2024.tar.gz
  3.	Navigate to CVSROOT Directory:
      cd Repositories/CVSROOT
  4.	Edit Configuration Files:
      o	Edit config:
        vim config
        # Add or uncomment the line:
        # UseNewInfoFmtStrings=yes
      o	Edit passwd:
        vim passwd
        # Add the following line:
        <username>:efzvTkvuxLOoA:cvs
  5.	Set Ownership and Permissions:
      chown -Rf cvs:cvs /home/cvs/Repositories
      chmod -Rf 775 /home/cvs/Repositories

Step 5: Install Additional Package
  1.	Install httpd-tools:
      yum install httpd-tools
  2.	Using cvs passwd login
      cd /home/cvs/Repositories/CVSROOT
      htpasswd passwd <username>
      vim passwd
      output :   <username>:encryped_password
      Add this entry in passwd file.
      here we might need to modify the file like <username>:encryped_password:cvs

Step 6: Configure xinetd for CVS
  1.	Navigate to xinetd Configuration Directory:
      cd /etc/xinetd.d/
  2.	Create and Edit cvspserver:
      vim cvspserver
      # Add the following content:
      service cvspserver
      {
          disable     = no
          port        = 2401
          socket_type = stream
          protocol    = tcp
          wait        = no
          user        = root
          passenv     = PATH
          server      = /usr/bin/cvs
          env         = HOME=/home/cvs
          server_args = -f --allow-root=/home/cvs/Repositories pserver
          bind        = <ServerIPaddress>
      }

Step 7: Start and Enable xinetd Service
  1.	Start xinetd Service:
      systemctl start xinetd.service
  2.	Enable xinetd Service on Boot:
      systemctl enable xinetd.service
  3.	Check xinetd Service Status:
      systemctl status xinetd.service

Step 8: Verify CVS Server
  1.	Check if cvspserver is Listening:
      netstat -l | grep cvspserver
  2.	Check for Port 2401 Specifically:
      netstat -ltnp | grep :2401

Step 9: Set CVSROOT Environment Variable
  1.	Navigate to Root's Home Directory:
      cd /root
  2.	Edit .bash_profile to Set CVSROOT:
      vim .bash_profile
        # Add the following line:
        export CVSROOT=:pserver:cvs@192.168.1.115:2401/home/cvs/Repositories

Step 10: Install Git and Clone Migration Script
  1.	Install Git:
      yum install git -y
  2.	Clone the CVS-Git Migration Repository:
      git clone https://github.com/kumaresangit/cvs-git-migration.git
  3.	Navigate to the Cloned Repository:
      cd cvs-git-migration
  4.	Copy the Migration Script Archive:
      cp cvs2svn-trunk.tar.gz ../.
  5.	Extract the Migration Script Archive:
      tar -xvzf cvs2svn-trunk.tar.gz

Step 11: Convert CVS Repositories to Git
  1.	Navigate to the Extracted Directory:
      cd cvs2svn-trunk/
  2.	Create Temporary Directory for Conversion:
      mkdir cvs2git-tmp
  3.	Run the CVS to Git Conversion:
      python cvs2git --blobfile=cvs2git-tmp/git-blob.dat --dumpfile=cvs2git-tmp/git-dump.dat --username=cvs2git /home/cvs/Repositories/ --fallback-encoding=UTF-8
  4.	Initialize a Bare Git Repository:
      git init --bare /root/Repositories.git
  5.	Navigate to the Git Repository:
      cd /root/Repositories.git/
  6.	Import Blob Data into Git:
      git fast-import --export-marks=../cvs2svn-trunk/cvs2git-tmp/git-marks.dat <../cvs2svn-trunk/cvs2git-tmp/git-blob.dat
  7.	Import Dump Data into Git:
      git fast-import --import-marks=../cvs2svn-trunk/cvs2git-tmp/git-marks.dat <../cvs2svn-trunk/cvs2git-tmp/git-dump.dat
  8.	Run Git Garbage Collection:
      git gc --prune=now

Following these steps will set up a CVS server on CentOS 7, migrate CVS repositories to Git, and verify that the server is listening on the correct port.












