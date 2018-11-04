<?php
/**
 * FRONT OFFICE TELEGRAM
 *
 * An open source application development framework for PHP
 *
 * This content is released under the MIT License (MIT)
 *
 * Copyright (c) 2018 e-project-Technology
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy
 * of this software and associated documentation files (the "Software"), to deal
 * in the Software without restriction, including without limitation the rights
 * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 * copies of the Software, and to permit persons to whom the Software is
 * furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in
 * all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
 * THE SOFTWARE.
 *
 * @package	Telegram Front Office
 * @author	Harjito
 * @copyright	E-Project Technology (https://blog.e-project-tech.com/)
 * @license	https://opensource.org/licenses/MIT	MIT License
 * @since	Version 1.0.0
 * @filesource
 */

class Telegram{

	public $con;
	public $chat;
	public $id;
	public $part;
	public $data;
	public $first_name;
	public $last_name;
	public $message;
	public $admin = '';//id telegram penjawab pesan 
  public $target;
  public $apikey = '';//api key telegram

	function __construct($con, $chat)
	{
		$this->con = $con;
		$this->chat = $chat;
		$this->id = $chat['chat']['id']; 
		$this->first_name = $chat['chat']["first_name"];
		$this->last_name = $chat['chat']["last_name"];
		$this->target = $this->id;
        
		$this->part = explode(' ',preg_replace("/\s+/",' ', strtolower($chat['text']))); 
		
		$query = "select * from bot where user_id= '".$this->id."'";
		$res = $con->query($query);	
		
		if($res->num_rows ==0){
			$this->data  = false;
		}else{
			$this->data  = $res->fetch_object();
		}
	}

    function check_command()
    {
        $ar_command = [
            'identity'=>'identity',
            'complain'=>'complain',
            'start'=>'greeting',
            'help' => 'help',
            'schedule' => 'schedule',
        ];
        
        $reset_status = [
            'identity'=>false,
            'complain'=>false,
            'start'=>true,
            'help' => true,
            'schedule' => true,
        ];

        $this->target = $this->id;

		$command = str_replace('/','',$this->part[0]);
		
        if( preg_match("/reply/", $command) || preg_match("/reply/", $this->data->command) )
        {
            if(preg_match("/reply/", $command))
            {
                $this->set_status($command);
                $this->data->command = $command;
            }
            return $this->reply();
        }
    
        if( $this->data->command == null || $reset_status[$command] )
        {
            if($reset_status[$command])
            {
                $this->unset_status();
            }
            else if($reset_status[$command]===FALSE)
            {
                $this->set_status($command);
            }
            
            if($this->part[1] != '' || $reset_status[$command])
            {
                
                if(method_exists($this, $ar_command[$command])){
                    return $this->{$ar_command[$command]}($this->part[1]);
                }
            
                return $this->unregister_command();
                
            }
            
            if(method_exists($this, 're_'.$ar_command[$command])){
                return $this->{'re_'.$ar_command[$command]}($this->part[1]);
            }

            return $this->unregister_command();
            
        }
        
        if(method_exists($this, $ar_command[$this->data->command]))
        {                
            return $this->{$ar_command[$this->data->command]}();
        }
            
        return $this->unregister_command();
        
    }

    function identity($param='')
    {
        if($this->data->identity != null){
            return $this->identity_exists();
        }
        
        if($this->get_identity($param)->num_rows == 0){
            $this->message = "Sdr. {$this->first_name}. NIM $param tidak ada dalam sistem.";
            return $this->sendMessage();    
        }

        $query = "update bot set identity = '{$param}', command = NULL where user_id='{$this->id}'";
        $this->con->query($query);
        $this->message = "Sdr. {$this->first_name}. NIM $param berhasil didaftarkan.";
        return $this->sendMessage();

    } 

    function re_identity()
    {
        if($this->data->identity != null){
            return $this->identity_exists();
        }

        $this->message = "Sdr. {$this->first_name}. Ketikkan NIM anda.";
        return $this->sendMessage();
            
    } 

    function identity_exists()
    {
        $this->unset_status();            
        $this->message = "Akun anda sudah didaftarkan untuk NIM {$this->data->identity}.";
        $this->message .= "\nJika itu bukan NIM anda silahkan ajukan complain.";
        $this->message .= "\n/complain pesan yang ingin disampaikan.";
        $this->sendMessage();
    }

    function get_identity($nim)
    {
        $query = "select nim from peserta where nim = '{$nim}'";
        return $this->con->query($query);
    }

    function get_client($user_id)
    {
        $query = "select * from bot where user_id = '{$user_id}'";
        return $this->con->query($query);
    }

    function unregister_command()
    {
        $this->target = $this->id;
        $this->message = "Sdr. $this->first_name. Perintah yang anda masukkan tidak ada dalam daftar.";
        $this->sendMessage();
    }

	function register()
	{
        $this->target = $this->id;
        
        $query = "insert into bot (user_id,first_name,last_name) values('{$this->id}','{$this->first_name}','{$this->last_name}')";
        
        if($this->con->query($query))
        {
            $this->message = "Halo $this->first_name. Selamat bergabung di telegram UAS Jurusan Kimia UNNES.";
        	$this->message .= "\nAgar anda dapat mengakses info seputar ujian silahkan daftarkan NIM anda dengan cara ketikkan";
			$this->message .= "\n/identity NIM_ANDA";
            return $this->sendMessage();
        }

        $this->message = "Halo $this->first_name Pendaftaran anda gagal diproses.";
		$this->message .= "Silahkan keetikan /start untuk mengulangi.";
        $this->sendMessage();       	
	}

    function set_status($command)
    {
        $query = "update bot set command = '{$command}' where user_id='{$this->id}'";
        $this->con->query($query);
    }

    function unset_status()
    {
        $query = "update bot set command = NULL where user_id='{$this->id}'";
        $this->con->query($query);
    }

    function greeting()
    {
        if($this->data->user_id  == null)
        {
            $this->message = "Halo $this->first_name. Selamat bergabung dengan kami.";
            $this->message .= "Silahkan  daftarkan NIM anda untuk mendapatkan informasi seputar uas.";
            return $this->sendMessage();
        }

        if($this->data->identity  == null){
            $this->message = "Halo $this->first_name. Selamat bergabung kembali denga kami.";
            $this->message .= "Silahkan  daftarkan NIM anda untuk mendapatkan informasi seputar uas.";
            return $this->sendMessage();
        }

        $this->message = "Halo $this->first_name. Selamat bergabung kembali denga kami.";
        $this->sendMessage();
    }

    function help()
    {
        $this->unset_status();
        $this->message = "Halo $this->first_name. Perintah yang tersedia saat ini adalah \n/help : melihat daftar perintah.";
	    $this->message .= "\n/Identity: mendaftarkan NIM.";
	    $this->message .="\n/schedule : melihat jadwal.";
        $this->message .= "\n/complain : mengajukan komplain.";
	    $this->sendMessage();    
    }

    function schedule()
    {
        $this->unset_status();
        $this->message = "Sdr. $this->first_name. Kami akan mencarikan informasi terkait data jadwal ujian anda.";
	    $this->message .= "\n Mohon tunggu untuk beberapa saat..";
	    $this->sendMessage();

        $query="SELECT j.kode, p.nama, m.nama as mata_uji,m.sks,h.tanggal,s.jam,r.nama as ruang, d1.nama as awas1, d2.nama as awas2 FROM peserta p ";
        $query.="INNER JOIN bot b on p.nim = b.identity LEFT JOIN jadwal j ON p.kode=j.kode LEFT JOIN plot l on j.kode=l.jadwal LEFT JOIN matkul m on j.mata_uji=m.kode ";
        $query .="LEFT JOIN hari h on h.ke=l.hari LEFT JOIN sesi s on l.sesi=s.ke LEFT JOIN  ruang r on r.kode=l.ruang ";
        $query .= "LEFT JOIN dosen d1 on d1.kode=l.awas1 LEFT JOIN dosen d2 on d2.kode=l.awas2 where b.user_id='".$res['id']."' order by h.ke,s.ke";
        
        $result=$this->con->query($query);
        if($result->num_rows ==0){
            $this->message = "Sdr. $this->first_name. Anda tidak memiliki agnda ujian.";
            return $this->sendMessage();
        }

        $this->message = "Sdr. $this->first_name. Daftar ujian anda sebagai berikut.";
	    $i = 1;
	    while($obj = $result->fetch_object()){
		    $this->message .= "\n$i. Mata Uji: ".$obj->mata_uji;
            $this->message .= "\nPelaksanan: ".($obj->tanggal==null ? "belum diset" : $obj->tanggal) .". jam: ".($obj->jam == null ? "belum diset" : $obj->tanggal);
            $this->message .= "\nRuang: ".($obj->ruang == null ? "belum diset" : $obj->ruang);
            $this->message .= "\nPengawas: ".($obj->awas1 ==null ? "Belum diset" : $obj->awas1).($obj->awas2 == null ? '' : " dan ".$obj->awas2);
		    $i++;
	    }
	    $this->sendMessage();
	
    }

    function complain()
    {
        if($this->data->method == null)
        {
            $this->set_status('complain');
        }
        if($this->part[0] == 'complain' || $this->part[0] == '/complain'){
            unset($this->part[0]);
        }

        $complain = str_replace('_', " ", implode(' ', $this->part));
        
        if($complain == '')
        {
            return $this->re_complain();
        }
        $this->message = "Sdr. $this->first_name. Keluhan anda sudah saya sampaikan kepada panitia. Silahkan tunggu paling lambat 1 x 24 jam.";
        $this->sendMessage();
        
        $query = "select p.* from peserta p RIGHT JOIN bot b on b.identity = p.nim where b.user_id='{$this->id}'";
	    $result = $this->con->query($query);
	    $obj = $result->fetch_object();

	    $this->message = "sdr. $this->first_name dengan identitas NIM: ".$obj->nim.", nama: ".$obj->nama." menyampaikan komplain bahwa:";
        $this->message .= "\n$complain";
        
        $repl = "\n/reply_".$this->encrypt($this->id);
        $this->target = $this->admin;
		$this->sendMessage($repl);
		$this->logger(['user_id'=>$this->id, 'type'=>'complain', 'content'=>$complain]);
        $this->unset_status();    
    }

    function re_complain()
    {
        $this->message = "Sdr. $this->first_name. Silahkan sampaikan keluhan anda.";
        $this->sendMessage();
    }

    function debug($message)
    {
        $this->message = json_encode($message);
        $this->sendMessage();
    }

    function encrypt($string)
    {
        $ori = str_split( (string) $string);
        $insert = str_split(substr(str_shuffle('abcdefghijklmnopkrstuvwxyz0123456789'),0,strlen($string)));
        $merge = [];
        foreach($insert as $i=>$v){
            $merge[] = $v;
            $merge[] = $ori[$i];
        }
        return implode('',$merge);
    }

    function decrypt($string)
    {
        $split = str_split($string,2);
        $output = '';
        foreach ($split as $i=>$v) {
            $output .= substr($v,-1,1);
        }
        return $output;
    }
    
    function reply()
    {
        $user_id = $this->decrypt((explode('_',$this->data->command))[1]);

        $client = $this->get_client($user_id);

        if(str_replace('/','',$this->part[0]) == $this->data->command)
        {
            unset($this->part[0]);
        }

        
        if(! isset($this->part[0])){
            return $this->re_reply($client);
        }
        $reply = implode(' ', $this->part);
        $obj = $client->fetch_object();
        $this->target = $user_id;
        $this->message = "Sdr. $obj->first_name, $reply. Terima kasih atas partsipasinya.";
        $this->sendMessage();
        $this->target = $this->id;
		$this->message = "Pesan untuk Sdr. $obj->first_name sudah terkirim.";
		
		$this->logger(['user_id'=>$user_id, 'type'=>'reply', 'content'=>$reply, 'officer'=>$this->id]);
        
        $this->sendMessage(' ');
        $this->unset_status();
    }

    function re_reply($client)
    {
        $obj = $client->fetch_object();
        $this->message = "Sampaikan tanggapan untuk pengaduan Sdr. $obj->first_name.";
        $this->sendMessage();
    }

	function logger($log)
	{
		if($this->data->command != null)
		{
			$log['datetime'] = date('Y-m-d H:i:s'); 
			$path = realpath('../log/') . '/' .$log['user_id'].'.log'; 			
			if(file_exists($path))
			{
				return @file_put_contents($path, json_encode($log)."\n", FILE_APPEND );
			}
			@file_put_contents($path, json_encode($log)."\n" );
		}
	}

	function sendMessage($rep = '')
	{
		if($rep == ''){
			$sig ="\nUntuk mengetahui perintah apa saja yang tersedia ketikan /help";
		}else{
			$sig = $rep;
		}
        $sendto = "https://api.telegram.org/bot".$this->apikey."/sendmessage?parse_mode=html&chat_id=".$this->target."&text=".urlencode($this->message.$sig);
		return file_get_contents($sendto);
	}	
}

require_once("data_con.php");
$content = file_get_contents("php://input");
$update = json_decode($content, true);

$telegram = new Telegram($mysqli, $update["message"]);
if($telegram->data ==false){
    return $telegram->register();
}
$telegram->check_command();
