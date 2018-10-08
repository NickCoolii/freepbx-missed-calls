# freepbx-missed-calls
Astersik + FreeBPX + Missed Calls

# Instruction
DANGER! It's really messy.

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
$sql = "SELECT `calldate`, `disposition` FROM `cdr` WHERE linkedid = '{$agi->request['agi_uniqueid']}' AND `src` = '{$agi->request['agi_callerid']}' and lastapp = 'Dial' GROUP BY `disposition` ORDER BY `calldate` DESC;";
$results = $db2->sql($sql, ASSOC);

$isAnswered = false;

foreach ($results as $result) {
	if ($result['disposition'] == 'ANSWERED') {
		$isAnswered = true;
		break;
	}
}

if (!$isAnswered) {
	$text .= "Missed call ☎️ {$agi->request['agi_callerid']} at " . date('H:i:s') . "\nCallback!"; # $result['calldate'] - nope
	sendMsg($text);
}
```

sendMsg() - function for sending messages from `messenger.php`
