# freepbx-missed-calls
Astersik + FreeBPX + Missed Calls at Queues

# Instruction
DANGER! It's really messy.

Relies on call recording (which calls AGI `attendedtransfer-rec-restart.php`) and CDR.

Add at end of file `/var/lib/asterisk/agi-bin/attendedtransfer-rec-restart.php` next code:

`include_once('/var/lib/asterisk/agi-bin/call-check.php');`

Add into the class `AGIDB` (file `sql.php`) next code
```php
public function set_db($dbname) {
  $this->dbname = $dbname;
}
```

Content of file `call-check.php`:
```php
<?php

include_once('phpagi.php');
include_once('sql.php');
include_once('messenger.php');

$agi = new AGI();

$db2 = new AGIDB($agi);
$db2->set_db('asteriskcdrdb');
$sql = "SELECT `disposition` FROM `cdr` WHERE linkedid = '{$agi->request['agi_uniqueid']}' AND `src` = '{$agi->request['agi_callerid']}' and `lastapp` = 'Queue' AND `disposition` = 'ANSWERED';";
$results = $db2->sql($sql, ASSOC);

$noAnswer = false === $results[0];

if ($noAnswer) {
	$text .= "Missed call ☎️ {$agi->request['agi_callerid']} at " . date('H:i:s') . "\nCallback!";
	sendMsg($text);
}
```

sendMsg() - function for sending messages from `messenger.php`
