Timinyto
========

Projects

<?php
       session_start();
          
      if(!isset($_SESSION["manager"]))
       {
  	   header("location: ../login.php");
		   exit();
	   }
       $loaner_id = $_SESSION["id"] ;
        $rate = $_SESSION["rate"];
        $duration = $_SESSION["duration"];
       // $username = $_SESSION["username"];
        $amount_invested = $_SESSION["amount_invested"];
     
        
           include $_SERVER['DOCUMENT_ROOT'] . '/includes/db.inc.php';
                 $result = mysqli_query($link,"SELECT id, username, amount_applied ,category,rate,duration,
                  date_added,loan_balance FROM Loanees WHERE rate = '$rate' AND duration = '$duration' AND loan_balance > 0");
		                 if (!$result)
                         {
                            $error = 'Error fetching loanees: ' . mysqli_error($link);
                            include 'error.html.php';
                            exit();
                          }
                          
                  while ($row = mysqli_fetch_array($result))
                   {
					   
                        $loanees[] = array('id'=> $row['id'],'username' => $row['username'],'amount_applied'=>$row['amount_applied'], 'rate' =>$row['rate'], 'duration'=>$row['duration'],
                          'date_added'=>$row['date_added'],'loan_balance'=> $row['loan_balance'] );
				   }
              //if (!isset($_SESSION['loan']))
                if (!isset($_SESSION['cart']))
                  {
                     $_SESSION['cart'] = array();
                    // $_SESSION['loan']
                  }
                  
                   if (isset($_POST['action']) and $_POST['action'] == 'Allot')
                {
                  // Add item to the end of the $_SESSION['cart'] array
                  /*$date = date('Y-m-d');*/
                  // $time = date(d M Y H:i:s);
                  $time = strftime('%c');
                  //echo "$time";
                  $_SESSION['cart'][] = $_POST['id'];
                  $id_no =  $_POST['id'];
                  foreach($_SESSION['cart'] as $id){
					  if($id == $id_no){
					   $x++;
					   if($x > 1){
						 //  echo 'twice';
						 $error = 'loanee already alloted';
						   array_splice($_SESSION['cart'] , $i-1, (count($_SESSION['cart'] ) - ($i-1))) ; 
							  break;
					   }
				   }
				}
                  
                  
                  $allotment = 10000 ;
                  //$_SESSION['cart'][] = $_POST['id'];
              //    $id  =  $_POST['id'];
                 // $_SESSION['cart'][] = $_POST['id'];
                  //echo "$id";
                  foreach ($_SESSION['cart'] as $id)
                   {
					  /* while(list($key, $value) = each($item)){
					   echo "$key, $value";
				   }*/
					  // echo "$quantity";
					 //++$quantity;
					   ++$i ;
						$till = $allotment * $i ;
						if( $till > $amount_invested)
						  {
							  $error = 'MAXIMUM AMOUNT ALLOTED FURTHER ALLOTMENT IS IMPOSSIBLE PLEASE SELECT ANOTHER LOANER';
							   array_splice($_SESSION['cart'] , $i-1, (count($_SESSION['cart'] ) - ($i-1))) ; 
							  break;
						  }
						  ///////////////////////////////
						  //to remove loanees from the session cart array
			           /////////////////////////////////
				   }
                  //header('Location:catalogue.html.php');
                //  exit();
                } 
               
                  if (isset($_POST['action']) and $_POST['action'] == 'Unallot all >>>')
                   {
                // Empty the $_SESSION['cart'] array
                     unset($_SESSION['cart']);
                     header('Location: ?cart');
                     exit();
                   }
                   
                     if (isset($_GET['cart']))
                {
                 $cart = array();
                  $total = 0;
                  $allotment = 10000 ;
                   foreach ($_SESSION['cart'] as $id)
                    {
                      foreach ($loanees as $product )
                       {
                          if ($product['id'] == $id)
                           {
							    
							   ++$quantity;
							   ++$x;
                             $cart[] = $product;
                              $total += $product['loan_balance'];
                              $product['quantity'] += $quantity ; 
                              $till = $allotment  * $x;
                              if($till >= $amount_invested)
                               {
								   $till = $amount_invested ;
								   
							   }
                              break;
                           }
                       }
                  }
             include 'cart.html.php'; 
             exit();
           }
           
           if(isset($_GET['end']))
            {
				$date = date('Y-m-d');
				  $time = strftime('%c');
				 $allotment = 10000 ;
				foreach($_SESSION['cart'] as $id)
				{
					foreach($loanees as $loanee)
					{
					if($id == $loanee['id'])
					  {
						++$x ;  
						 $till = $allotment  * $x;
						 
				      $total += $loanee['loan_balance'];
				      $new_loanbalance = $loanee['loan_balance'] - $allotment ;
				      $investment_balance = $amount_invested - $till;
				    //  echo "$new_loanbalance , $investment_balance";
				    //$rate = $row['rate'];
		            //$duration = $row['duration'];
		            //$loan_balance =  $row['balance'];
		            $loan_balance = ($allotment + ($allotment * ($rate/100))*($duration/12));
				    ////////////
				      include $_SERVER['DOCUMENT_ROOT'] . '/includes/db.inc.php';
				   //updates the loaners and loanees accounts on clicking select another loner
				   
				   $sql = "INSERT INTO  investments SET  amount = '$allotment' , rate = '$rate' , duration = '$duration' ,  balance= '$loan_balance' ,  loaner_id= '$loaner_id' , loanee_id='$id' , 
				                 date_added = '$date' , time_added = '$time' ";
				   $result = mysqli_query($link , $sql);
				   if(!isset($result)){
					     $error =  'error creating investment';
					     include 'error.html.php';
                         exit();
				   }
				   $sql= "INSERT INTO  loans SET  amount = '$allotment' , rate = '$rate' , duration = '$duration' ,  balance= '$loan_balance' ,  loanee_id= '$id',  loaner_id = '$loaner_id'  ,
				                 date_added = '$date' , time_added = '$time' ";
				  // $resulty = ($link , "INSERT INTO `loans`.`loans` (`amount`, `rate `, `duration`,`balance`,`loanee_id`) VALUES ( '$allotment','$rate','$duration', '$allotment' , '$id' )");
				   $result = mysqli_query($link , $sql);
				  
				   if(!isset($result)){
					   $error =  'error updating loanee and loaner balances';
					    include 'error.html.php';
                         exit();
				   }
				   
				   //to create actual loan contract
				    $sql = "UPDATE Loanees INNER JOIN loaners   SET loan_balance='$new_loanbalance' , investment_balance = '$investment_balance'  
				                  WHERE Loanees.id = '$id' AND loaners.id = '$loaner_id' ";  
				    $result = mysqli_query($link , $sql);
				  
				   if(!isset($result)){
					   $error =  'error updating loanee and loaner balances';
					    include 'error.html.php';
                         exit();
				   }
				   //include $_SERVER['DOCUMENT_ROOT'] . '/includes/db.inc.php';
				 $sql = "SELECT id FROM loans WHERE loanee_id = '$id' AND loaner_id = '$loaner_id' AND time_added = '$time' ";
				 $result = mysqli_query($link, $sql);
				 $row = mysqli_fetch_array($result);
				 $loan_id = $row['id'];
				// while($row = mysqli_fetch_array($result)){
				 /*$loan_id[] = array();
				 $loan_id[] = array( 'id' => $row['id']); 
				  //echo "$loan_id,,";           
			     }
			     foreach($loan_id as $id){
				$id = $id['id'];	 
			}*/
				 $sql = "SELECT id FROM investments WHERE loaner_id = '$loaner_id' AND loanee_id = '$id' AND time_added = '$time' ";
				 $result = mysqli_query($link, $sql);
				 $row = mysqli_fetch_array($result);
				 $investment_id = $row['id'];           
				// echo "$investment_id";
				 
				  $sql = " INSERT INTO   invested_loans SET amount = '$loan_balance', loan_id = '$loan_id' , investment_id = '$investment_id'  " ;
				  $result = mysqli_query($link, $sql);
				  //echo "$result";
				  if(!isset($result)){
					  $error = 'error entering the invested loans into the invested loans table';
					  include 'error.html.php';
					  exit();
				  }
				   
				  $sql = " INSERT INTO  interest_rate SET rate = '$rate', loan_id = '$loan_id' , investment_id = '$investment_id'  " ; 
				     $result = mysqli_query($link, $sql);
				  if(!isset($result)){
					  $error = 'error entering the invested loans into the invested loans table';
					  include 'error.html.php';
					  exit();
				  }
				  ///////////from above
				       /*$sql = "SELECT loans.id , investments.id FROM loans INNER JOIN investments WHERE loanee_id = '$id' AND loaner_id = '$loaner_id' "; 
				                 $result = mysqli_query($link , $sql);
				                
				                 while($row = mysqli_fetch_array($result)){
									  $investment_id = row['investments.id'];
									  $loan_id = row['loans.id'];
								 
								 $sql = " INSERT INTO   invested_loans SET amount = '$allotment', loan_id = '$loan_id' , investment_is = '$investment_id'  " ;
								 
								 $sql = " INSERT INTO  interest_rate SET rate = '$rate', loan_id = '$loan_id' , investment_is = '$investment_id'  " ;     
								  }
				        */                
				    
				      unset($_SESSION['cart']);             
				     break;
				      }
				   }
				 }
				 //echo "$total";
				header("Location: transact.html.php");
				exit();
			}
?>
 <?php include_once $_SERVER['DOCUMENT_ROOT'] .
  '/includes/helpers.inc.php'; ?>
  <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
   "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
 <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
   <head>
    <title>Loans catalog</title>
    <meta http-equiv="content-type"
    content="text/html; charset=utf-8" />
      <link href="../../main/style.css" type="text/css" rel="stylesheet">
    <style type="text/css">
  
   td, th {
   border: 1px solid black;
   }
   </style>
   </head>
  <body>
	  <div id="wrapper">
	 <div id= "container" >
       <div id= "header" align="center">
       <div id="logo"><img src="../../logo.jpg" width=100px height=100px></div>
       <h1>YICHOUSE PEER LENDING</h1>
       </div>	
                  <a href='../logout.html.php' style='float:right;'><button value='Sign Out'>Sign Out</button></a>
               <div id="topNav"><nav><ul id="navigation-bar"><li><a href="../../home/">HOME</a>
      </li>
       <li>
	     <a href="../../admin/">ADMIN</a>
       </li>
      <li>
	   <a href="../../getloan/">GET LOAN</a>
     </li>
     <li><a href="../../makeinvestment/">MAKE INVESTMENT</a>
     </li>
     <li><a href="#">ABOUT US</a>
     </li>
     <li><a href="#">CONTACT US</a>
     </li></ul></nav></div>
        <div id="mainContent">"
        <p align="center" style="color: red ; font-size: 16px;"><?php echo "$error" ;?></p>
      <p align="center">&nbsp;&nbsp;You have alotted the loanees investment to <a href="#" style="color:orange; font-size:16px;"><?php
       echo count($_SESSION['cart']); ?> </a> loanees already. Amount available for allotment is
       <a href= "#"style="color: orange; font-size:16px;">KSHS.<?php echo "$amount_invested"; ?> </a>that should not be exceeded.
       Every loanee is alloted equal amount of <a href="#" style="color: red; text-decoration: none; font-size:16px">KSHS.10,000</a></p>
       
       <p align='center' style='font-size: 16px;'>You should allot investment to only <a href='' style='color: red;'>ONE LOANEE </a> at a time</p>
       
<p align="center" ><a href="?cart" style="color:orange;text-decoration:none; ">View the loanees </a>already issued a loan to</p>
<p align='center' ><a href="?end" style="color: orange;" onclick="confirm('are you sure you would like to commit these loans?')">
	COMMIT the loans here </a>and select another loaner to allot investment </p>
    <center>
		 
     <table border="1">
      <thead>
      <tr>
       <th>Username</th>
       <th>Amount applied</th>
       <th>Rate(%)</th>
        <th>Duration(months)</th>
       <th>Date_added</th>
       <th>Loan Balance(Kshs)</th>
      
      </tr>
    </thead>
    <tbody>
     <?php foreach ($loanees as $loanee): ?>
     <tr>
      <td><?php htmlout($loanee['username']); ?></td>
      <td>
       <?php echo htmlout($loanee['amount_applied']); ?>
      </td>
       <td><?php htmlout($loanee['rate']); ?></td>
        <td><?php htmlout($loanee['duration']); ?></td>
      <td><?php htmlout($loanee['date_added']); ?></td>
      <td><?php htmlout($loanee['loan_balance']); ?></td>
      
      <td>
		  
      <form action="" method="post"  onsubmit="confirm('are you sure you want to allot this loanee?')">
       <div>
        <input type="hidden" name="id" value="<?php
          htmlout($loanee['id']); ?>"/>
        <input type="submit" name="action" value="Allot" />
       </div>
     </form>
     </td>
      </tr>
    <?php endforeach; ?>
   </tbody>
   </table>
   </center>
    <!--<p align="center"><a href="..">Return to Home</a></p>-->
    </div>
    <div>
      <?php
     include '../static-footer/footer.inc.html.php' ;
    ?>
   </div>
   </div>
 </div>
   </body>
 </html>


