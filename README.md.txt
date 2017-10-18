user: inovasi
pass: inovasi

user: root
pass: root


MASTER
IP 192.168.56.102

STANDBY
IP 192.168.56.103
--setting parameter MASTER
cd /var/lib/pgsql/9.5/data

vi postgresql.conf

listen_addresses='*'
wal_level = hot_standby
max_wal_senders = 3
wal_keep_segments = 32
hot_standby = on

vi pg_hba.conf
host     replication     postgres        192.168.56.103/32       trust
host     replication     postgres        192.168.56.102/32       trust

--stop & start database
pg_ctl stop
pg_ctl start

--STANDBY
cd /var/lib/pgsql/9.5
mkdir data
chmod 700 data

pg_basebackup -P -R -X stream -c fast -h 192.168.56.102 
-U postgres -D data/

cd /var/lib/pgsql/9.5/data

vi recovery.conf
recovery_target_timeline = 'latest'
restore_command = 'cp /var/lib/pgsql/9.5/data/pg_xlog/%f %p'
trigger_file = '/var/lib/pgsql/9.5/data/failover.txt'

--start standby
pg_ctl start

select pg_last_xlog_receive_location() "receive_location",
pg_last_xlog_replay_location() "replay_location",
pg_is_in_recovery() "recovery_status";


--SWITCHOVER
--stop MASTER
pg_ctl stop

--SLAVE
--convert slave to new master
touch /var/lib/pgsql/9.5/data/failover.txt

--MASTER (New Standby 192.168.56.102)
--create recovery.conf
--start database
pg_ctl start

--test replikasi


--SWITCH BACK from Master 192.168.56.103 
to Slave 192.168.56.102

--SLAVE
--disable replication
select pg_xlog_replay_pause();

--enable replication
select pg_xlog_replay_resume();






















