# patientForms_superadmin.php

<?php
  session_start();
  require("classes/err.php");
  require_once("classes/CTherapists.php");
  require_once("classes/CPatients.php");
  require_once("classes/CResponses.php");
  require_once("classes/CForms.php");
  require_once("scripts/common.php");
  require_once("classes/CAgencies.php");
  require_once("scripts/CheckUrl.php");
  require_once("classes/CommonClass.php");
  global $TI_1;
?>
<script type="text/javascript" src="//cdn.jsdelivr.net/fetch/2.0.1/fetch.min.js"></script>
<script type="text/javascript" src="//cdn.jsdelivr.net/es6-promise/3.1.2/es6-promise.js"></script>
<script type="text/javascript" src="//cdn.pubnub.com/sdk/javascript/pubnub.4.4.2.js"></script>
<script type="text/javascript" src="//cdn.rawgit.com/onsip/SIP.js/0.7.7/dist/sip-0.7.7.js"></script>
<script type="text/javascript" src="//cdn.rawgit.com/ringcentral/ringcentral-js/3.1.0/build/ringcentral.js"></script>
<script type="text/javascript" src="emradmin/js/ringcentral/ringcentral-web-phone.js"></script>
<!--<script type="text/javascript" src="js/rc/custom-rc.js"></script>-->
<script type="text/javascript" src="../emradmin/js/ringcentral/custom-rc.js"></script>
<script type="text/javascript" src="js/dropzone.js"></script>

<script>
$('#rc-sms').on('hidden.bs.modal', function () {
                      $('#msgerr').text('');
                      $('#btnsendsms').attr('disabled',false);
                      $('#sms-loader').hide();
                      $('#hidsmsphone').val('');
                      $('#sms-history').html('');
                  });
              
              
                  $('#divsmstoall').on('shown.bs.modal', function () {
                      $('#phone_loader').show();
                      $('#msgerrall').text('');
                      $('#btnsendsms').attr('disabled',false);
                      $('#smsall-loader').hide();
                      $('#txtmessageall').val('');                      
                      $('#messageall_status').text('');
                      
                      $.ajax({
                        type:"get",
                        url:"common_functions.php",
                        data:({'func':'getTherapistsPhone','status':1}),
                        success:function(e){
                          $('#thphone_list').html(e);
                          $('#phone_loader').hide();
                        }
                      });
                  });
function makeSms_popup(call,n){
                $('#hidsmsphone').val(call);
                $('#rc-sms').modal('show');
                $('#smsheader').text(n);
                getSMSHistory(call,'<?php echo date('Y-m-d',  strtotime('-3 days')); ?>');
              }
              
              function makeSMS(){
                var $phone = $('#hidsmsphone').val();
                var $message = $('#txtmessage').val().trim();
                if($message==''){
                    $('#msgerr').text('Enter message!');
                }
                else{
                    $('#msgerr').text('');
                    $('#btnsendsms').attr('disabled',true);
                    $('#sms-loader').show();
                    
                    send_sms($phone,$message);
//                    $.ajax({
//                      'type':'post',
//                      'url':'ringcentral_api.php',
//                      data:({'func':'send_sms','phone':$phone,'message':$message}),
//                      success:function(e){
//                        $('#btnsendsms').attr('disabled',false);
//                        $('#sms-loader').hide();
//                        $('#msgerr').text(e);
//                      }
//
//                    });
                }
              }

</script>
<div id="rc-sms" class="modal fade" role="dialog">
                <div class="modal-dialog modal-lg">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h4>Send SMS to <span id="smsheader"></span></h4>
                        </div>
                        <div class="modal-body">
                            <label>Last Conversation:</label>
                            <div id="sms-history" class="sms-history">
                              
                            </div>
                            <form id="frm_sms">
                              <div class="form-group">
                                  <label>Message:</label>
                                  <textarea class="form-control" id="txtmessage" name="txtmessage"></textarea>
                                  <label id="msgerr" class="text-red"></label>
                              </div>         
                                
                              <input type="hidden" name="hidsmsphone" id="hidsmsphone">
                            </form>
                        </div>
                        <div class="modal-footer margin-top-0">
                            <div id="sms-loader" style="display: none;" class="pull-left">
                                <img src="resources/images/ajax-loader.gif" /> Processing request, please wait...
                            </div>
                            <label id='message_status'></label>
                            <button class="btn btn-primary" id="btnsendsms" type="button" onclick="makeSMS();">Send SMS</button>
                            <button type="button" id="btnmodelclose" class="btn btn-danger btn-simple" data-dismiss="modal">Close</button>
                        </div>
                    </div>
                </div>            
            </div>
<?php
  $curAgency = new CAgencies();
  $objCommonClass = new CommonClass();
  #error_reporting(E_ALL); ini_set('display_errors',1);
  //Sidebar menu active tab
  $_SESSION["activeTab"] = "theraPatients";

  if($_SESSION['therapistId']==''){
   $_SESSION['therapistId'] = 0;
  }

  //User Validation
  $userid = 0;
  if (isset($_SESSION["userId"]) && $_SESSION["userId"] != "") {  
      $userid = $_SESSION["userId"];
  }
  else{
    header("Location: login.php?err_id=-3" );
  }

  $referralId = $_GET["referralId"];

  $curPatient = new CPatients();
  
  //Patient Details
  $row = $curPatient->GetPatientDetailsByRefId($referralId); 
  
  //Referral Details
  $row_refer = $curPatient->getReferralsDetailById($referralId);
  $agency_data =  $curAgency->GetAgencyDetails($row_refer['AgencyId']);
  
  $therapist_id = $row_refer['TherapistId'];
  $patient_id = $row_refer['PatientId'];
  
  $curTherapist = new CTherapists();
  $therapist_data = $curTherapist->GetTherapistDetails($row_refer['TherapistId']);
  $therapist_phone = $therapist_data['Phone'];
  $therapist_name = $therapist_data['LastName']." ".$therapist_data['FirstName'];
  
//pt need oasis or not
  $oasis_status = $row_refer['is_oasis'];

  //Cert Expired or not
  $soc_date = date("m-d-Y",strtotime($row_refer['soc_date']));
  $soc_update = date("Y-m-d",strtotime("+59 days",strtotime($row_refer['soc_date'])));
  if($soc_update <= date("Y-m-d"))
    $soc_expired = 'Yes';
  else
    $soc_expired = 'No';

  //set patient's session
  $patientName = $row["FirstName"] . " " . $row["LastName"];
  $patientMRNo = $row["MRNo"];
  $patientId = $row["Id"];

  $_SESSION["selectedPatientId"] = $row["Id"];
  $_SESSION["patientName"] = $patientName;
  $_SESSION["patientMRNo"] = $patientMRNo;
  $_SESSION["AgencyId"] = $row_refer['AgencyId'];
  $_SESSION["ReferPhysician"] = $row["ReferPhysician"];
  
  
  //if other therapist try to access other therapist's patient note
  if ($_SESSION["therapistId"] != $row_refer['TherapistId'] && $_SESSION["userType"] == 1) {  
    header("Location: theraPatients.php");
  }

  //Function for count 
  function get_visit_array($total_week, $start_date, $A, $B, $C, $D, $E, $F, $G, $H) {
  $j = 0;
  $visit_array = array();
  $start = "";
  $end = "";
  $visits = "";
  $day = date('N', strtotime($start_date));
  $start_date;
  switch ($day) {
    case 1:
      $total_day = 5;
      break;
    case 2:
      $total_day = 4;
      break;
    case 3:
      $total_day = 3;
      break;
    case 4:
      $total_day = 2;
      break;
    case 5:
      $total_day = 1;
      break;
    case 6:
      $total_day = 0;
      break;
    case 7:
      $total_day = 6;
      break;
  }

  $start = $start_date;
  $end = date('Y-m-d', strtotime("+" . $total_day . "day" . $start_date));
  if ($B != "" && $B != 0) {

    for ($i = 1; $i <= $B; $i++) {
      if ($i != 1) {
        $start = date('Y-m-d', strtotime("+1day" . $end));
        $end = date('Y-m-d', strtotime("+7day" . $end));
      }
      if ($i == 1) {
        $start = $start_date;
        $end = date('Y-m-d', strtotime("+" . $total_day . "day" . $start_date));
      }
      $j++;
      $visit_array[$j]['visits'] = $A;
      $visit_array[$j]['start'] = $start;
      $visit_array[$j]['end'] = $end;
    }
  }

  if ($D != "" && $D != 0) {
    for ($i = 1; $i <= $D; $i++) {
      $start = date('Y-m-d', strtotime("+1day" . $end));
      $end = date('Y-m-d', strtotime("+7day" . $end));
      $j++;
      $visit_array[$j]['visits'] = $C;
      $visit_array[$j]['start'] = $start;
      $visit_array[$j]['end'] = $end;
    }
  }

  if ($F != "" && $F != 0) {
    for ($i = 1; $i <= $F; $i++) {
      $start = date('Y-m-d', strtotime("+1day" . $end));
      $end = date('Y-m-d', strtotime("+7day" . $end));
      $j++;
      $visit_array[$j]['visits'] = $E;
      $visit_array[$j]['start'] = $start;
      $visit_array[$j]['end'] = $end;
    }
  }

  if ($H != "" && $H != 0) {
    for ($i = 1; $i <= $H; $i++) {
      $start = date('Y-m-d', strtotime("+1day" . $end));
      $end = date('Y-m-d', strtotime("+7day" . $end));
      $j++;
      $visit_array[$j]['visits'] = $G;
      $visit_array[$j]['start'] = $start;
      $visit_array[$j]['end'] = $end;
    }
  }
    
  
  
//  if($_SERVER['REMOTE_ADDR']=='43.228.96.30'){
//    echo "<pre>"; print_r($visit_array);exit;    
//  }
  #
  
  
 #
  $curAgency = new CAgencies();
  $query = "SELECT group_concat(r.Id ORDER BY r.Id ASC) AS rId FROM tblreferrals r JOIN tblusers u WHERE r.TherapistId = u.TherapistId AND u.UserType = 1 AND r.PatientId = " . $_SESSION['selectedPatientId'];

  $find_referral = $curAgency->GetDataList($query);

  if ($find_referral)
    $resFind = $find_referral->fetch_assoc();
  
  foreach ($visit_array as $item) {
    $date = date('Y-m-d', strtotime("+1day" . $item['end']));
    $datetime = $date . " 12:00:00";
    
    $pt_week =  date('m/d/Y', strtotime($item['start'])) . " - " . date('m/d/Y', strtotime($item['end']));
    $planned_visit = $item['visits'];
    $done_visits = 0;
    
    if (time() > strtotime($datetime)) {      
      
      $query = "SELECT COUNT(*) as cnt FROM tbl_dvr WHERE referral_id IN(" . $resFind['rId'] . ")  AND is_submitted = 'yes' AND dvr_date BETWEEN '" . $item['start'] . "' AND '" . $item['end'] . "'";
      $DVR = $curAgency->GetDataList($query);

      $q = "SELECT COUNT(*) as spof_count FROM tblSPOF WHERE ReferralId IN(" . $resFind['rId'] . ") AND "
          ."IF(PO_10!='',DATE_FORMAT(STR_TO_DATE(PO_10, '%m/%d/%Y'),'%Y-%m-%d'),DATE_FORMAT(STR_TO_DATE(PO_16, '%m/%d/%Y'),'%Y-%m-%d')) BETWEEN '" . $item['start'] . "' AND '" . $item['end'] . "'";
      $SPOF = $curAgency->GetDataList($q);

      if ($SPOF)
        $SPOFFind = $SPOF->fetch_assoc();
      
      if ($DVR)
        $DVRFind = $DVR->fetch_assoc();
      
      $done_visits = $DVRFind['cnt'];
      if ($DVRFind['cnt'] < $item['visits']) {        
        
          $pt_visit_array = array('status'=>1,'pt_week'=>$pt_week,'planned_visit'=>$planned_visit,'done_visit'=>$done_visits);
          $visits = 1;
          return $pt_visit_array;
          
      } elseif (($DVRFind['cnt'] > $item['visits']) && $SPOFFind['spof_count'] <= 0) {        
        
          $pt_visit_array = array('status'=>2,'pt_week'=>$pt_week,'planned_visit'=>$planned_visit,'done_visit'=>$done_visits);
          $visits = 2;
          return $pt_visit_array;
          
      } else {        
        
          $pt_visit_array = array('status'=>0,'pt_week'=>$pt_week,'planned_visit'=>$planned_visit,'done_visit'=>$done_visits);
          $visits = 0;      
          
      }
    } else {    
          $pt_visit_array = array('status'=>0,'pt_week'=>$pt_week,'planned_visit'=>$planned_visit,'done_visit'=>$done_visits);
          $visits = 0;      
    }
  }
  if($visits==0){
      return $visits;
  }
}

  // Allow Eval Or Not case of 
  // Start Code for check patient assigne before 5 days and not submiited any DVR , CC note and SPOF note.
  $allow_eval = '';

  //Check Patient Is Transfered or not
  #error_reporting(E_ALL); ini_set('display_errors', 1);
  $old_ref_id = $objCommonClass->get_first_referral_id($patientId,1);

  if($old_ref_id == $referralId){  
    // 
    $current_date_1 = date('Y-m-d');
    $no_dvr_sql = "SELECT COUNT(dvr.id) AS total_dvr, DATEDIFF('".$current_date_1."',  DATE(ref.DateSubmitted)) as date_diff
            FROM tblreferrals AS ref 
            LEFT JOIN tbl_dvr AS dvr ON dvr.referral_id = ref.Id        
            WHERE (DATEDIFF('".$current_date_1."',DATE(ref.DateSubmitted)) > 5) AND 
            ref.deleteFlag = 0 AND ref.dischargeFlag = 0 AND ref.StatusId = 2 AND ref.Id = $referralId
            GROUP BY ref.Id HAVING COUNT(dvr.id) = 0";
    $no_dvr_data = $curAgency->execute_query($no_dvr_sql);

    if(!empty($no_dvr_data)){

      $no_cc_spof_where = "ReferralId = ".$referralId." AND StatusId = 2 AND (cancel_flag = '' OR cancel_flag IS NULL OR cancel_flag = 'no')";
      $cc_count = $curAgency->selectData('tblCCNote',array('COUNT(*) as total_cc'),$no_cc_spof_where);
      $spof_count = $curAgency->selectData('tblSPOF',array('COUNT(*) as total_spof'),$no_cc_spof_where);
      if($cc_count[0]->total_cc <= 0 && $spof_count[0]->total_spof <= 0){
        $allow_eval = 'Please complete CC form or SPOF form to report reason for delayed Evaluation visit.';
      }
    }

  }
 // End Code for check patient assigned before 5 days and not submiited any DVR , CC note and SPOF note.



/* code start for checking visits completed status based on frequency */

/* find all previouse and current referrals of this patient */

$ref_ids = 0;
$referral_ids_sql = "SELECT group_concat(r.Id ORDER BY r.Id ASC) AS rId FROM tblreferrals r JOIN tblusers u WHERE r.TherapistId = u.TherapistId AND u.UserType = 1 AND r.PatientId = " . $_SESSION['selectedPatientId'];
$find_referral = $curAgency->GetDataList($referral_ids_sql);
if ($find_referral){
  $resFind = $find_referral->fetch_assoc();
  $ref_ids = $resFind['rId'];
}

/* ----------- */

/* Find frequency and date of last POC form */

$last_poc_sql = "SELECT E.TI_1,p.TI_W_3,p.TI_1_a AS A, p.TI_1_b AS B, p.TI_1_c AS C, p.TI_1_d AS D, p.TI_1_e AS E, p.TI_1_f AS F, p.TI_1_g AS G, 
p.TI_1_h AS H FROM tblPOC p JOIN tblEval E WHERE p.ReferralId IN($ref_ids) AND E.Id = p.EvalId ORDER BY p.Id DESC LIMIT 1";

$find_POC = $curAgency->GetDataList($last_poc_sql);
if ($find_POC){
  $resFindPOC = $find_POC->fetch_assoc();  
  $last_poc_date = $resFindPOC['TI_1'];
}

/* check if any MV form submitted after POC submitted with FREQ */
/* Changes 05-06-2017 check SPOF form submiited */
    //Missed Visit
    $able_to_submit_discharge = 'no';
    $resFindMV = array();
    $MV_Fields = array('PO_1','A','B','C','D','E','F','G','H','total_week','effective_date','visit_date');
    $MV_Where = "visit_date > '$last_poc_date' AND referralId IN($ref_ids) AND form_status = 'submitted' AND cancel_flag = 'no'";    
    $MV_data = $curAgency->selectData('tblmissedvisit',$MV_Fields,$MV_Where,'ORDER BY id DESC LIMIT 1');
    $mv_date = '';
    $today_date = date('Y-m-d');
    if(!empty($MV_data)){
      
      if($MV_data[0]->PO_1=='No New Orders given'){
        $able_to_submit_discharge = 'yes';
        // check second last form
        $MV_Fields = array('PO_1','A','B','C','D','E','F','G','H','total_week','effective_date','visit_date');
        $MV_Where = "visit_date > '$last_poc_date' AND referralId IN($ref_ids) AND form_status = 'submitted' AND cancel_flag = 'no' AND PO_1 != 'No New Orders given'";        
        $MV_data = $curAgency->selectData('tblmissedvisit',$MV_Fields,$MV_Where,'ORDER BY id DESC LIMIT 1');
        if(!empty($MV_data)){
          if(strtotime($MV_data[0]->effective_date) && strtotime($MV_data[0]->effective_date) >= strtotime($last_poc_date) && strtotime($MV_data[0]->effective_date)<= strtotime($today_date)){
            $resFindMV = (array)$MV_data[0];
            #echo '<pre>'; print_r($resFindMV); exit;
            $mv_week = $resFindMV['total_week'];
            $mv_effective_date = $resFindMV['effective_date'];
            $mv_A = $resFindMV['A']; 
            $mv_B = $resFindMV['B'];
            $mv_C = $resFindMV['C'];
            $mv_D = $resFindMV['D']; 
            $mv_E = $resFindMV['E'];
            $mv_F = $resFindMV['F'];
            $mv_G = $resFindMV['G'];
            $mv_H = $resFindMV['H'];
            $mv_date = $resFindMV['visit_date']; 
          }
        }
      }
      else if(strtotime($MV_data[0]->effective_date) && strtotime($MV_data[0]->effective_date) >= strtotime($last_poc_date) && strtotime($MV_data[0]->effective_date)<= strtotime($today_date)){
        $resFindMV = (array)$MV_data[0];
        #echo '<pre>'; print_r($resFindMV); exit;
        $mv_week = $resFindMV['total_week'];
        $mv_effective_date = $resFindMV['effective_date'];
        $mv_A = $resFindMV['A']; 
        $mv_B = $resFindMV['B'];
        $mv_C = $resFindMV['C'];
        $mv_D = $resFindMV['D']; 
        $mv_E = $resFindMV['E'];
        $mv_F = $resFindMV['F'];
        $mv_G = $resFindMV['G'];
        $mv_H = $resFindMV['H'];
        $mv_date = $resFindMV['visit_date'];        
      }
    }

    //SPOF
    $spof_fields = array('PO_8','A','B','C','D','E','F','G','H','PO_10','total_visit_continue','total_week_continue','PO_11','PO_14','A1','B1','C1','D1','E1','F1','G1','H1','PO_16','total_visit1','total_week1',"date_format(str_to_date(PO_29, '%m/%d/%Y'), '%Y-%m-%d') as PO_29");
    $spof_where = "(PO_11 = 1 OR PO_8 = 1 OR PO_14 = 1) AND str_to_date(PO_29, '%m/%d/%Y') >= '$last_poc_date' AND referralId IN($ref_ids) AND StatusId = 2 AND (cancel_flag = 'no' || cancel_flag IS NULL)";
    $spof_data = $curAgency->selectData('tblSPOF',$spof_fields,$spof_where,'ORDER BY id DESC LIMIT 1');
    $spof_hold_data = array();
    $spof_date = $spof_status = '';
    
    if(!empty($spof_data)){
      
      if($spof_data[0]->PO_11 == 1 && strtotime($spof_data[0]->PO_13) && strtotime($spof_data[0]->PO_13) > strtotime($last_poc_date) && strtotime($spof_data[0]->PO_13)<= strtotime($today_date)){ //hold
        $able_to_submit_discharge = 'yes';
        $spof_status = "hold";       
        $spof_date = $spof_data[0]->PO_13;
        $spof_effective_date = $spof_data[0]->PO_13;
        
      }
      else if($spof_data[0]->PO_8 == 1 || $spof_data[0]->PO_14 == 1){
        
        if($spof_data[0]->PO_8 == 1 && strtotime($spof_data[0]->PO_10) && strtotime($spof_data[0]->PO_10) >= strtotime($last_poc_date) && strtotime($spof_data[0]->PO_10)<= strtotime($today_date)){ // continue
            $spof_status = "continue";
            $spof_week = $spof_data[0]->total_week_continue;
            $spof_effective_date = $spof_data[0]->PO_10;
            $spof_A = (int)$spof_data[0]->A;
            $spof_B = (int)$spof_data[0]->B;
            $spof_C = (int)$spof_data[0]->C;
            $spof_D = (int)$spof_data[0]->D;
            $spof_E = (int)$spof_data[0]->E;
            $spof_F = (int)$spof_data[0]->F;
            $spof_G = (int)$spof_data[0]->G;
            $spof_H = (int)$spof_data[0]->H; 
            $spof_date = $spof_data[0]->PO_10;
        }
        else if(strtotime($spof_data[0]->PO_16) && strtotime($spof_data[0]->PO_16) > strtotime($last_poc_date) && strtotime($spof_data[0]->PO_16)<= strtotime($today_date)) { //Resume
            $spof_status = "resume";
            $spof_week = $spof_data[0]->total_week1;
            $spof_effective_date = $spof_data[0]->PO_16;          
            $spof_A = (int)$spof_data[0]->A1;
            $spof_B = (int)$spof_data[0]->B1;
            $spof_C = (int)$spof_data[0]->C1;
            $spof_D = (int)$spof_data[0]->D1;
            $spof_E = (int)$spof_data[0]->E1;
            $spof_F = (int)$spof_data[0]->F1;
            $spof_G = (int)$spof_data[0]->G1;
            $spof_H = (int)$spof_data[0]->H1;
            $spof_date = $spof_data[0]->PO_16;
            //check previos SPOF is hold or not
            $spof_hold_fields = array('PO_11',"date_format(str_to_date(PO_29, '%m/%d/%Y'), '%Y-%m-%d') as PO_29");
            $spof_hold_where = "(PO_11 = 1 OR PO_8 = 1 OR PO_14 = 1) AND str_to_date(PO_29, '%m/%d/%Y') < '{$spof_data[0]->PO_29}' AND referralId IN($ref_ids) AND StatusId = 2 AND (cancel_flag = 'no' || cancel_flag IS NULL)";
            $spof_hold_data = $curAgency->selectData('tblSPOF',$spof_hold_fields,$spof_hold_where,'ORDER BY id DESC LIMIT 0,1');
            #echo 'rest<pre>'; print_r($spof_hold_data); exit;
        }
        //$spof_date = $spof_data[0]->PO_29;
      }
    }
    
    $freq_note = "";
    $therapist_will_visit = array();
    if($spof_status!='' && $spof_status!='hold' && !empty($resFindMV)){
      if(strtotime($spof_effective_date) >= strtotime($mv_effective_date)){ //Freq. of SPOF
        
        $freq_note = "SPOF Note1 - ".date('m/d/Y',strtotime($spof_effective_date));        
        $visit_array = get_visit_array($spof_week,$spof_effective_date,$spof_A,$spof_B,$spof_C,$spof_D,$spof_E,$spof_F,$spof_G,$spof_H);
        
        //Get total visit that Therapist will do.
        $total_visit = ($spof_A * $spof_B) + ($spof_C * $spof_D) + ($spof_E * $spof_F) + ($spof_G * $spof_H);
        $therapist_will_visit = array('effective_date'=>$spof_effective_date,'total_visit'=>$total_visit);
        
      }
      else{ //Missed Visit Freq.
        $freq_note = "MV Note1 - ".date('m/d/Y',strtotime($mv_effective_date));
        $visit_array = get_visit_array($mv_week,$mv_effective_date,$mv_A,$mv_B,$mv_C,$mv_D,$mv_E,$mv_F,$mv_G,$mv_H);
        
        //Get total visit that Therapist will do.
        $total_visit = ($mv_A * $mv_B) + ($mv_C * $mv_D) + ($mv_E * $mv_F) + ($mv_G * $mv_H);
        $therapist_will_visit = array('effective_date'=>$mv_effective_date,'total_visit'=>$total_visit);  
      }
      
    }
    else if($spof_status!='' && $spof_status!='hold' ){ //Freq. of SPOF      
      #echo '<pre>'; print_r($spof_data); exit;
      $freq_note = "SPOF Note2 - ".date('m/d/Y',strtotime($spof_effective_date));
      $visit_array = get_visit_array($spof_week,$spof_effective_date,$spof_A,$spof_B,$spof_C,$spof_D,$spof_E,$spof_F,$spof_G,$spof_H);
      
      //Get total visit that Therapist will do.
      $total_visit = ($spof_A * $spof_B) + ($spof_C * $spof_D) + ($spof_E * $spof_F) + ($spof_G * $spof_H);
      $therapist_will_visit = array('effective_date'=>$spof_effective_date,'total_visit'=>$total_visit);
    }
    else if(!empty($resFindMV) && $mv_week!=0 && $mv_week!=''){ //Missed Visit Freq.
      $freq_note = "MV Note2 - ".date('m/d/Y',strtotime($mv_effective_date));
      $visit_array = get_visit_array($mv_week,$mv_effective_date,$mv_A,$mv_B,$mv_C,$mv_D,$mv_E,$mv_F,$mv_G,$mv_H);
      
      //Get total visit that Therapist will do.
      $total_visit = ($mv_A * $mv_B) + ($mv_C * $mv_D) + ($mv_E * $mv_F) + ($mv_G * $mv_H);
      $therapist_will_visit = array('effective_date'=>$mv_effective_date,'total_visit'=>$total_visit);
    }
    else{ //POC Freq.  
      $freq_note = "POC Note - ".date('m/d/Y',strtotime($resFindPOC['TI_1']));
      $visit_array = get_visit_array($resFindPOC['TI_W_3'], $resFindPOC['TI_1'], $resFindPOC['A'], $resFindPOC['B'], $resFindPOC['C'], $resFindPOC['D'], $resFindPOC['E'], $resFindPOC['F'], $resFindPOC['G'], $resFindPOC['H']);
      
      //Get total visit that Therapist will do.
      $total_visit = ((int)$resFindPOC['A'] * (int)$resFindPOC['B']) + ((int)$resFindPOC['C'] * (int)$resFindPOC['D']) + ((int)$resFindPOC['E'] * (int)$resFindPOC['F']) + ((int)$resFindPOC['G'] * (int)$resFindPOC['H']);
      $therapist_will_visit = array('effective_date'=>$resFindPOC['TI_1'],'total_visit'=>$total_visit);
    }
  /* End Code */
    
    #echo "<pre>"; print_r($visit_array); exit;
    if($_SERVER['REMOTE_ADDR']=='43.228.96.54'){
    #echo "<pre>"; print_r($freq_note);exit;    
    }
    


  // START get Private Insurance Information
  $pvt_insu_sql = "SELECT p.*,(SELECT GROUP_CONCAT(TI_4 ORDER BY str_to_date(TI_4,'%m/%d/%Y') ASC SEPARATOR ', ') FROM tblNote WHERE StatusId = 2 AND cancel_flag = 'no' AND ReferralId IN ($ref_ids) AND pvt_insurance_id = p.id) as visit_details "
                . "FROM tbl_private_insurence as p where p.patient_id = $patientId AND p.referral_id IN ($ref_ids)"; 
  $pvt_insu_data = $curAgency->execute_query($pvt_insu_sql);
  #echo '<Pre>'; print_r($pvt_insu_data); exit;
  // END get Private Insurance Information


/* Get Latest Eval Date */
$query = "SELECT group_concat(r.Id ORDER BY r.Id ASC) AS rId FROM tblreferrals r JOIN tblusers u WHERE r.TherapistId = u.TherapistId AND u.UserType = 1 AND r.PatientId = " . $_SESSION['selectedPatientId'] . " AND r.deleteFlag = 0";

$find_referral = $curAgency->GetDataList($query);
if ($find_referral)
  $resFind = $find_referral->fetch_assoc();

$last_eval_date = $curAgency->selectData('tblEval', array('TI_1'), "ReferralId IN(" . $resFind['rId'] . ")", 'order by Id  DESC LIMIT 1');
  #  echo '<pre>'; print_r($last_eval_date); exit;
//End get Latest Eval


  //Start Code for SPOF and Cnote and re-eval
  /*
   * 1. SPOF submitted after POC 
   * 2. If HOLD, not allowed EVAL and CN
   * 3. If Resume, Allowed Eval not CN
   * 4. If Eval after Resume, allowed CN
   * 
   */  
    
  $spof_cn_alert = $re_eval_alert = $hold_message =  'no';
  
  //Check Is SPOF with Hold is Active or not
   if(($spof_status == "hold" || $spof_status == "resume") && strtotime($spof_effective_date) > strtotime($last_poc_date)){ 
    
    if($mv_effective_date == '' || ($mv_effective_date != '' && strtotime($spof_effective_date) > strtotime($mv_effective_date))){ 
      
      if($spof_status == "hold"){
        
        $spof_cn_alert = $hold_message = $re_eval_alert = 'yes';
      }
      else if(!empty($spof_hold_data) && $spof_status == "resume" && strtotime($spof_effective_date) > strtotime($last_eval_date[0]->TI_1)){
        
        $spof_cn_alert = 'yes';
        $re_eval_alert = 'no'; //Not allowed to CN
      }
      else{
        
        $spof_cn_alert = $re_eval_alert = 'no';
      }
      
    }
  } 
  //End Code for SPOF and Cnote and re-eval

  
  // Last Reassessment Date
  $last_reass_date = $curAgency->selectData('tbl30day', array('cVisitDate'), "ReferralId IN(" . $resFind['rId'] . ") AND cancel_flag!='yes'", 'order by Id  DESC LIMIT 1');
  if (isset($last_reass_date[0]->cVisitDate) && $last_reass_date[0]->cVisitDate != "") {

    $last_reass_date[0]->cVisitDate = date("Y-m-d", strtotime($last_reass_date[0]->cVisitDate));
    if ($last_reass_date[0]->cVisitDate > $last_eval_date[0]->TI_1) {

      $last_reassessment = $last_reass_date[0]->cVisitDate;
    } else
      $last_reassessment = $last_eval_date[0]->TI_1;
  }
  else if(isset($last_eval_date[0]->TI_1)){
    $last_reassessment = $last_eval_date[0]->TI_1;
  }
  else{
    $last_reassessment = '';
  }


$refer_id = "SELECT group_concat(r.Id) as ids FROM tblreferrals r JOIN tblusers u WHERE r.TherapistId = u.TherapistId AND u.UserType = 1 AND r.PatientId = " . $_SESSION['selectedPatientId'] . "";

$result_find_referral = $curAgency->GetDataList($refer_id);
if ($result_find_referral)
  $refer_ids = $result_find_referral->fetch_assoc();



$query = "SELECT count(Id) as total FROM tblPOMA WHERE ReferralId in (" . $refer_ids['ids'] . ") AND StatusId = 2";

#echo $query."<br/>";

$result = $curAgency->GetDataList($query);
if ($result)
  $row1 = $result->fetch_assoc();
$totalpoma = $row1['total'];
#echo "$totalpoma"; 
//GET ALL OASIS FORM
$_SESSION['referralId'] = $_GET['referralId'];




$referral_list = "SELECT group_concat(r.Id) as ids FROM tblreferrals r JOIN tblusers u WHERE r.TherapistId = u.TherapistId AND u.UserType = 1 AND r.PatientId = " . $_SESSION['selectedPatientId'] . "";

$result_find_referral = $curAgency->GetDataList($referral_list);
if ($result_find_referral)
  $setallflagfalse = $result_find_referral->fetch_assoc();

#echo $_SESSION['selectedPatientId']; exit;
$oasis_list = $curAgency->selectData('tbl_oasis', array('id', 'referralId', 'created_date', 'visit_date', 'ES_date', 'submitted_date', 'form_status', 'cancel_flag'), 'referralId IN (' . $setallflagfalse['ids'] . ')', 'ORDER BY id ASC');
$oasis_list_new = $curAgency->selectData('tbl_oasis_newversion', array('id', 'referralId', 'created_date', 'visit_date', 'ES_date', 'submitted_date', 'form_status', 'cancel_flag'), 'referralId IN (' . $setallflagfalse['ids'] . ')', 'ORDER BY id ASC');

#echo "<pre>"; print_r($oasis_list); exit;
$w = "referralId IN (" . $setallflagfalse['ids'] . ") AND form_status = 'submitted'";
$count_oasis_list = $curAgency->selectData('tbl_oasis', array('id'), $w);
$count_oasis_list_newversion = $curAgency->selectData('tbl_oasis_newversion', array('id'), $w);
#echo "<pre>"; print_r($count_oasis_list);
    



//END OASIS FORM LOGIC

/* -----------Visits Remaining Logic START----------------- */
 /* -----------
  * Visits Remaining Logic START 
  * Changed on 26-09-2017 By Viral 
  * new logic
  * total visit remaining = total visits from active(latest) freq from (POC/MV/SPOF) - submitted clin note after defining active freq.
  * ----------------- */
$total_visit_remaining = $run_visit_validation = 0; 
if($_SERVER['REMOTE_ADDR']=='43.228.96.57'){
  #echo '<pre>'; print_r($therapist_will_visit); exit;
}
if(!empty($therapist_will_visit)){
  if(strtotime($therapist_will_visit['effective_date'])){
    $effictive_date = date('Y-m-d',strtotime($therapist_will_visit['effective_date']));
    $total_visit = $therapist_will_visit['total_visit'];
    //get how many visits completed
    $count_cnote_where = "cancel_flag = 'no' AND StatusId = 2 AND ReferralId IN ({$setallflagfalse['ids']}) AND str_to_date(TI_4,'%m/%d/%Y') >= '$effictive_date'";
    $count_cnote_data = $curAgency->selectData('tblNote', array('COUNT(*) as total_note'), $count_cnote_where);
    $total_visit_remaining = $total_visit - $count_cnote_data[0]->total_note;
  }
  else{
 
    $run_visit_validation = 1;
  }
}
else{
  
  $run_visit_validation = 1;
}

if($_SERVER['REMOTE_ADDR']=='43.228.96.57'){
  #echo '<pre>'; print_r($total_visit_remaining); exit;
}

/* -----------Visits Remaining Logic END----------------- */

/* ----------- Nehal : Total Authorized visit Logic Start ----------- */
// Get ids of stop private insurance
$where_stop_private_auth      = "referral_id IN (" . $setallflagfalse['ids'] . ") AND status = 'Stopped'";
$stop_auth_visit_ids = $curAgency->selectData('tbl_private_insurence', array('GROUP_CONCAT(id) as stopcount'), $where_stop_private_auth);

//get how many visits completed
if($stop_auth_visit_ids[0]->stopcount != ''){
$where     = "cancel_flag = 'no' AND StatusId = 2 AND ReferralId IN (" . $setallflagfalse['ids'] . ") AND pvt_insurance_id NOT IN (" . $stop_auth_visit_ids[0]->stopcount . ")";
}else{
$where     = "cancel_flag = 'no' AND StatusId = 2 AND ReferralId IN (" . $setallflagfalse['ids'] . ")";
}
$data_note = $curAgency->selectData('tblNote', array('*'), $where);


$where_private_auth = "referral_id IN (" . $setallflagfalse['ids'] . ") AND status != 'Stopped'";
$total_visit_count_query = $curAgency->selectData('tbl_private_insurence',array('SUM(visit_authorized) as visit_authorized'),$where_private_auth);
$total_auth_visit = $total_visit_count_query[0]->visit_authorized;
//echo "<pre>";print_r($total_visit_count_query);exit;
$total_auth_visit_count = count($data_note);
if($_SERVER['REMOTE_ADDR']=='223.196.69.42'){
  #echo '<pre>'; print_r($total_visit_remaining); exit;
//  echo $total_auth_visit_count.'__'.$total_auth_visit;
}
/* ----------- Nehal : Total Authorized visit Logic End ----------- */


/* -----------due date calulation START----------------- */
$objAgency = new CAgencies();
$query = "SELECT rf.Id AS ReferralId, p.FirstName, p.LastName, t.LastName AS tlname, t.FirstName AS tfname, t.Email, t.Id, u.Id AS user_id,
          IF (b.cEval_VisitDate > a.cReass_VisitDate, b.cEval_VisitDate, a.cReass_VisitDate) last_visit_date,
          IF (b.cEval_VisitDate > a.cReass_VisitDate, b.cEval_VisitDate, a.cReass_VisitDate) + INTERVAL 30 DAY AS due_date
          FROM
            (
              SELECT b.PatientId, a.TI_1 AS cReass_VisitDate 
              FROM tblEval a,
              (
                SELECT r.PatientId, MAX(a.Id) AS max_eval_form_id
                FROM tblEval AS a, tblreferrals r
                WHERE a.StatusId ='2' AND r.Id = a.ReferralId GROUP BY r.PatientId 
              ) b
              WHERE a.Id = b.max_eval_form_id
            ) a 
          LEFT OUTER JOIN 
            (
              SELECT b.PatientId, DATE_FORMAT(STR_TO_DATE(a.cVisitDate, '%m/%d/%Y'),'%Y-%m-%d')  AS cEval_VisitDate
              FROM tbl30day a,
              (
                SELECT r.PatientId, MAX(a.Id) AS max_reass_form_id
                FROM tbl30day AS a, tblreferrals r
                WHERE a.StatusId ='2' AND r.Id = a.ReferralId GROUP BY r.PatientId 
              ) b
              WHERE a.Id = b.max_reass_form_id
            ) b       
          ON a.PatientId = b.PatientId
          JOIN tblreferrals AS rf ON rf.PatientId = a.PatientId 
          JOIN tblpatients AS p ON p.Id = a.PatientId
          JOIN tbltherapists AS t ON rf.TherapistId = t.Id
          JOIN tblusers AS u ON t.Id = u.TherapistId
          WHERE rf.activeTransferFlag = 1 AND rf.deleteFlag = 0 AND rf.StatusId = 2 AND p.Id=" . $_SESSION['selectedPatientId'] . " AND u.UserType = 1
          ORDER BY 1 DESC";
$res_eval = $objAgency->GetDataList($query);
$due_dt = '';
if ($row_eval = $res_eval->fetch_assoc()){  
  $due_dt = date("m/d/Y", strtotime($row_eval['due_date']));
}

if($due_dt){
  $_SESSION['reassment_date'] = $due_dt;
}

//GET Url

if($due_dt!='' && strtotime(date('m/d/Y'))>= strtotime($due_dt)) { 
   $get_where =  "ReferralId IN(". $setallflagfalse['ids'].") AND ".
        "StatusId = 2 AND DATE(date_format(str_to_date(TI_4, '%m/%d/%Y'), '%Y-%m-%d')) <= '".date('Y-m-d',strtotime($due_dt))."'";  
     
    $result_get_last_clin_note = $objAgency->selectData('tblNote',array('Id','TI_4 AS DateSubmitted'),$get_where,'ORDER BY Id desc LIMIT 0,1'); 
    #echo '<pre>'; print_r($result_get_last_clin_note); exit;
    if(!empty($result_get_last_clin_note)){
      $pt30_url = "pt30Day.php?referralId=".$_GET['referralId']."&evalId=&pocId=&noteId=".$result_get_last_clin_note[0]->Id."&date=&form=CNOTE";
    }
    else{
      $pt30_url = "pt30Day.php?referralId=".$_GET['referralId'];
    }
}


/* -----------due date calulation END----------------- */
//*******************************
//******ADDED BY CHIRAg For SHOW link UPLOAD DVR
////*******************************************
$querydvr = "SELECT e.TI_1,e.ReferralId,IF((SELECT COUNT(*) FROM tbl_dvr WHERE dvr_date = e.TI_1 AND referral_id=e.ReferralId) >0,'yes','no') AS dvr FROM tblEval e
        WHERE ReferralId=$referralId  AND cancel_flag='no' GROUP BY  e.TI_1
        UNION
        SELECT e.PI_2,e.ReferralId,IF((SELECT COUNT(*) FROM tbl_dvr WHERE dvr_date =STR_TO_DATE(e.PI_2,'%m/%d/%Y') AND  referral_id=e.ReferralId) >0,'yes','no') AS dvr FROM tbldischarge e
        WHERE ReferralId=$referralId AND cancel_flag='no' GROUP BY  e.PI_2
        UNION
        SELECT e.TI_4,e.ReferralId,IF((SELECT COUNT(*) FROM tbl_dvr WHERE dvr_date =  STR_TO_DATE(e.TI_4,'%m/%d/%Y') AND  referral_id=e.ReferralId) >0,'yes','no') AS dvr FROM tblNote e
        WHERE ReferralId=$referralId AND cancel_flag='no' GROUP BY  e.TI_4";
//$dvrupload_frm_data = $objAgency->GetDataList($querydvr);
//echo "here"; exit;

/** Check Reassessement Due Alert Enable or not **/
$disable_due_alert = 0;
if(isset($_SESSION['disabled_Reassess_Alert']) && !empty($_SESSION['disabled_Reassess_Alert'])){
  if(in_array($referralId,$_SESSION['disabled_Reassess_Alert'])){
      $disable_due_alert = 1;
  }
}

//Get Certification periods
$cert_periods = $objAgency->selectData('tblpatientcert_period',array('*'),"patient_id = $patientId AND therapist_type = '1'","ORDER BY id DESC");

/** End Check Reassessement Due Alert Enable or not **/

?>

<div class="modal fade stick-up" id="UploadDocumentModal" role="dialog" aria-labelledby="UploadDocumentModal" aria-hidden="true" data-backdrop="static" data-keyboard="false">
    <div class="modal-dialog">
        <div class="box" id="body-loader" style="display: none; z-index: 200; padding: 0; margin: 0; border: 0;" >
          <div class="overlay" style="position: fixed;"></div>
          <div class="loading-img" style="position: fixed;">
          </div>
        </div>
        <div class="modal-content">
            
            <input type="hidden" value="" name="UploadBy" id="UploadBy">
            <div class="modal-header clearfix ">
                <button type="button" class="close closeuploaddoc" data-dismiss="modal" aria-hidden="true"><i class="pg-close fs-14"></i>
                </button>
                <h4 class="p-b-5"><div class="modal-title">Upload Fax Document(s)</div></h4>
            </div>
            <div class="modal-body">
                <div id="fax-msg" style="padding: 10px; text-align: center; background-color: #d2f5cc; border: 1px solid #73c166; color: #165f0a; margin-bottom: 20px; border-radius: 4px; display: none">
                <i class="fa fa-check" aria-hidden="true" style="margin-right: 6px; font-size: 15px; position: relative; top: 1px;"></i>
                <span style="font-size: 15px; font-weight: 600;"> Document(s) uploaded successfully.</span>
                </div>
                <div class="alert alert-success" id="sucessdocument" style="display:none;" role="alert"></div>
                <div class="alert alert-danger" id="exceeded_storage_limit" role="alert" style="display: none;"></div>
                <form action="upload.php" class="dropzone" id="my-dropzone" style="background-color: #f7f7f7; border: 2px dotted rgba(0, 0, 0, 0.3) !important; padding: 10px;">
                    <input type="hidden" value='<?php echo $therapist_id; ?>' name='therapist_id' id="therapist_id">
                    <input type="hidden" value='<?php echo $patient_id; ?>' name='patient_id' id="patient_id">
                    <div class="fallback">
                    <input name="file" type="file" multiple/>
                    </div>
                </form>
                <div class="row">&nbsp;</div>
                
                <div class="modal-footer" style="padding: 20px 0 0; margin-top: 0;">
                    <button id="submit-all" class="btn btn-primary">Upload Document(s)</button>
                    <button id="cancel-all" type="button"  class="btn closeuploaddoc">Close</button>
                </div>
            </div>
        </div>
    </div>
</div>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="https://www.w3.org/1999/xhtml"><!-- InstanceBegin template="/Templates/masterPage.dwt.php" codeOutsideHTMLIsLocked="false" -->
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=iso-8859-1" />
        <!-- InstanceBeginEditable name="doctitle" -->
        <title>EMR</title>
        <!-- InstanceEndEditable -->
        <!-- InstanceBeginEditable name="head" -->
        <link rel="stylesheet" href="css/style.css"  type="text/css"/>
        <link rel="stylesheet" href="css/main.css"  type="text/css" />
        
        <!-- /* Bootstrap CSS */ --> 
        <link href="css/bootstrap.min.css" rel="stylesheet" type="text/css" />
        <link href="css/font-awesome.min.css" rel="stylesheet" type="text/css" />
        <link href="css/ionicons.min.css" rel="stylesheet" type="text/css" />
        <link href="css/AdminLTE.css" rel="stylesheet" type="text/css" />
        <link type="text/css" rel="stylesheet" id="arrowchat_css" media="all" href="https://www.airshometherapy.com/arrowchat/external.php?type=css" charset="utf-8" />
        <link href="css/dropzone.css" rel="stylesheet" type="text/css">
        <link href="https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css" rel="stylesheet">
        <!-- /* Bootstrap JS */ --> 
        <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.0.2/jquery.min.js"></script>
        <script src="https://ajax.googleapis.com/ajax/libs/jqueryui/1.10.3/jquery-ui.min.js" type="text/javascript"></script>

        <link rel="stylesheet" href="css/ui-lightness/jquery-ui-1.8.6.custom.css" type="text/css">        
        <script src="js/bootstrap.min.js" type="text/javascript"></script>
        <script src="js/AdminLTE/app.js" type="text/javascript"></script>
        
        
        
        <link rel="stylesheet" href="css/jquery.alerts.css"  type="text/css" />
        <script language="javascript" type="text/javascript" src="js/jquery.alerts.js"></script>
        
        <script language="JavaScript" src="js/common.js" type="text/javascript"></script>
        <script language="javascript" type="text/javascript" src="js/utilsPF.js"></script>
        

        <script language="JavaScript" src="js/jquery-ui-1.8.6.custom.min.js" type="text/javascript"></script>
        <script language="JavaScript"  type="text/javascript" src="js/customSelect.jQuery.js"></script>
        <script type="text/javascript" src="js/bootstrap-switch.min.js"></script>
        <link rel="stylesheet" href="css/bootstrap-switch.min.css"  type="text/css" />
        <script type="text/javascript" src="js/jquery.fancybox.js"></script>
        <script type="text/javascript" src="js/jquery.fancybox.pack.js"></script>
        <link rel="stylesheet" type="text/css" href="css/jquery.fancybox.css" media="screen" />
        <link rel="stylesheet" href="css/custom.css"  type="text/css" />
        <script type="text/javascript" src="js/cert_period.js"></script>
        <style type="text/css">
            .focusclass
            {
                color:#FF0000;
            }
            #fancybox-content {
                background-color: #E1EDC8; /* or whatever */
            } 
            .panelcontent
            {
                font-size:13px;
            }
        </style>
<style>
.switch {
  position: relative;
  display: inline-block;
  width: 48px;
  height: 23px;
}

.switch input {display:none;}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  -webkit-transition: .4s;
  transition: .4s;
}

.slider:before {
  position: absolute;
  content: "";
  height: 15px;
  width: 15px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  -webkit-transition: .4s;
  transition: .4s;
}

input:checked + .slider {
  background-color: #2196F3;
}

input:focus + .slider {
  box-shadow: 0 0 1px #2196F3;
}

input:checked + .slider:before {
  -webkit-transform: translateX(26px);
  -ms-transform: translateX(26px);
  transform: translateX(26px);
}

/* Rounded sliders */
.slider.round {
  border-radius: 34px;
}

.slider.round:before {
  border-radius: 50%;
}
</style>
        <script language="JavaScript"  type="text/javascript" src="js/customSelect.jQuery.js"></script>
        <script language="javascript" type="text/javascript">

          function check_visit_status(formId, obj){             
            window.location = $(obj).attr('url'); 
          }

          function dischargeLink(referralId) {
              // body...
              //alert(referralId);
              //check_visit_status(4);
              var totalpoma = Number('<?php echo $totalpoma; ?>');
              if (totalpoma > 0)
              {
                  if (confirm('Reminder to complete a Tinetti form prior to submitting Discharge'))
                      window.location.href = 'ptDischarge.php?referralId=' + referralId;
              } else {
                  window.location.href = 'ptDischarge.php?referralId=' + referralId;
              }


          }

        //$.noConflict();
          var flagConfilct = true;
          $(document).ready(function () {
              
              
                    
        $('.make-switch').bootstrapSwitch();

        $(document).on('switchChange.bootstrapSwitch','.make-switch', function(event, state) {
                var val = state ? 1 : 0;
                var record = $(this).data('record');
               
                $.ajax({
                                          type: 'POST',
                                          url: 'update_data_import.php',
                                          data: 'val=' + val + '&ref_id=' + '<?php echo $referralId; ?>',
                                          success: function (data) {
                                                                                        }
                                      });

                           });   
              
           var click1 = '<?php echo $row_refer['clin_note_auto_import'];?>';
           if(click1=='yes')
           {
              $("#import_switch").click(); 
           }
              

            <?php if($_SESSION["userType"]==0 || $_SESSION["userType"]==4) { ?>                            
                update_soc_date('<?php echo $referralId; ?>','<?php  echo date('Y-m-d',strtotime($soc_update));?>'); // Edited for samrat by nehal dor certi. date validation : 11-June-2018
            <?php } ?>
  
  
  <?php if(($disable_due_alert == 0 && $row_refer['StatusId']==2 && $row_refer['dischargeFlag']!=1 && $_SESSION["userType"] == 1)  && $due_dt!='' && strtotime(date('m/d/Y'))>= strtotime($due_dt)) { ?>
      $.alerts.okButton = ' Yes ';
      $.alerts.cancelButton = ' No ';
      jConfirm("Reassessesment is currently due. Would you like to complete it now?", 'Reassessesment Due', function(r)
      {
          var totalpoma1 = Number('<?php echo $totalpoma; ?>');
          if(r==true){
            if(totalpoma1==1){
              tinetti2('valid');
            }
            else{
              check_visit_status(5,'#btnreassessment');
            }

          }
          else{
              $.ajax({
                 'type':'GET',
                 'url':'commonAjax.php',
                 'data':({func:'disable_reasses_due_alert','referral_id':'<?php echo $referralId; ?>'}),
                 'success':function(e){
                    console.log('success');
                 }
              });
          }
      });
    <?php } ?>
      
      
          });
        // datepicker code by Rahul Gupta 11-1-2014 
          function tinetti()
          {
              $.alerts.okButton = '&nbsp;&nbsp;Proceed&nbsp;&nbsp;&nbsp;&nbsp;';
              $.alerts.cancelButton = '&nbsp;&nbsp;&nbsp;Cancel&nbsp;&nbsp;';
              jConfirm('Attention!  System wont let you SUBMIT poc form unless you submit at least one Tinetti form.\nIf you proceed now, you can just SAVE poc form.', '', function (answer) {
                  if (answer) {
                      window.location = "ptPOC.php?referralId=<?php echo $referralId; ?>";
                  } else {
                  }
              });
          }
          function tinetti2(a)
          {

              $.alerts.okButton = '&nbsp;&nbsp;Proceed&nbsp;&nbsp;&nbsp;&nbsp;';
              $.alerts.cancelButton = '&nbsp;&nbsp;&nbsp;Cancel&nbsp;&nbsp;';
              jConfirm('Attention!  System wont let you SUBMIT  Reassessment form unless you submit at least two Tinetti forms.\nIf you proceed now, you can just SAVE Reassessment form.', '', function (answer) {
                  if (answer) {
                    if(a=='valid'){
                      window.location = '<?=$pt30_url?>';
                    }
                    else{
                      window.location = "pt30Day.php?referralId=<?php echo $referralId; ?>";
                    }
                      //window.location = "pt30Day.php?referralId=< ?php echo $referralId; ?>";
                  } else {

                  }
              });
          }
          window.onload = function () {
              var div_effect = document.getElementById("loadingEffect");
              div_effect.style.display = "none";
          }
          
          
  function patient_note() {

       $.fancybox({
           'transitionIn' : 'elastic',
           'transitionOut'  : 'elastic',        
           'overlayShow'  : true,    
           width : 400,                    
           autoSize : true,
           iframe : {
               scrolling : 'no'
           },
           'href': "addpatientnote.php",
           'type': "iframe",
       });
   }
  function show_notes_by_cert_period(cert_id,pt_id,cert_start,cert_end){
     <?php if($oasis_status=='checked'){ ?>
        $is_oasis = "yes";
     <?php } else { ?>
        $is_oasis = 'no';
     <?php } ?>
       
     $.fancybox.open({
        href: "common_functions.php",
        type: "ajax",
        width:600,
        autoSize: false,
        ajax: {
            type: "get",
            data: {'func':'pt_notes_by_cert_period','patient_id':pt_id,'cert_id':cert_id,'cert_start':cert_start,'cert_end':cert_end,'is_oasis':$is_oasis}
        },
        beforeShow: function(){
          $(".fancybox-skin").css("backgroundColor","#999999");
        }
    });     
     
  }
        </script>
        <!-- InstanceEndEditable -->
    </head>
    <body class="skin-blue">
<?php require_once("scripts/header2.php") ?>

<?php require_once("scripts/sidebar.php") ?>
        <aside class="right-side">
            <section class="content-header">
                <h1 class="inline">
                    Patient Forms
                    <small>List of forms available for patient</small>
                </h1>
                <?php if($oasis_status == "checked"){ ?>
                <h1 style="color:red;display: inline-block;margin: 0 0 0 10px;">DC OASIS Needed</h1>
                <?php } ?>                
                <div class="pull-right">                    
                  <?php if ($_SESSION["userType"] == 0 || $_SESSION["userType"] == 4) { ?>
                    <a href="editPatientinfo.php?patientId=<?=$patientId?>&refId=<?=$referralId?>" style='' class="sign btn btn-success btn-sm">Edit Patient</a>
                    <button class="sign btn btn-success btn-sm" onclick='patient_note();' value='note'>Paitent Note</button>
                    <a  <?php if ($oasis_status == "checked") { ?> onClick="return Dvr_upload(<?php echo $referralId ?>, 'dc_oasis');" <?php } else { ?> onClick="return Dvr_upload(<?php echo $referralId ?>, 'ndcoasis');" <?php } ?> href="" style='' class="sign btn btn-success btn-sm"><i class="fa fa-upload"></i> Click to Upload DVR</a>

                  <?php } ?>
                </div>
            </section>
            
            <section class="content">
                <div class="row">
                                         
                        <div class="col-md-12">
                            <div class="box">
                                <div class="box-body">
                                <div class="box-header">
                                <div style="color:white;background-color:red;padding-left:10px;border-radius:2px;" id="notice"></div>    
                                    
                                  <?php if ($soc_expired == "Yes" && !isset($_SESSION['upadated_cert_per']) && ($_SESSION["userType"]==0 || $_SESSION["userType"]==4)) { ?>
                                  <div style="clear: both; display:inline-block; margin: 7px 0 0; text-align: right;width: 100%;">
                                      <a style="color:#fff;" href="javascript:void(0);" onclick="update_certification_period('<?php echo $referralId; ?>');" class="btn btn-success btn-xs">Update Cert Period</a>
                                  </div>
                                  <?php } ?>
                                        
                                  <div class="alert alert-warning custom-alert pull-left" style="width:100%;">
                                      <div class="row">
                                        <div class="col-sm-4">
                                            <label>Patient Name : <?php echo $patientName; ?></label>
                                        </div>
                                        <div class="col-sm-2">
                                          <label>MR No. : <?php echo $patientMRNo; ?></label>
                                        </div>
                                        <div class="col-sm-6 text-right certification-from">
                                            <div class="inline patient-form-list">
                                              <strong>Certification Period From :</strong> 
                                              <input type="text" readonly="readonly" id="txtSoc" value="<?php echo $soc_date; ?>" name="txtSoc" maxlength="100" class="form-control input" />
                                               <span>To</span>
                                               <input type="text" readonly="readonly" id="txtSoc1" value="<?php echo date('m-d-Y', strtotime($soc_update)); ?>" name="txtSoc1" maxlength="100" class="form-control input"  />
                                            </div>
                                        </div>                                   
                                      </div>      
                                  </div>
                                <span style="color:blue;">Therapist: <a href="editTherapist.php?therapistId=<?=$row_refer['TherapistId']?>" style="color:blue;"><?=$therapist_name?></a></span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                                <span style="color:blue;">Agency: <a style="color:blue;" href="editAgency.php?agencyId=<?=$row_refer['AgencyId']?>"><?=$agency_data['Title']?></a></span><br><br>
                                <a class="btn btn-sm btn-info margin-right-10" style="float:right" href="dashboard.php">Back</a>
                                <a class="btn btn-info" target="_blank" href="unpaired_dvr_list.php?therapistId=<?=$row_refer['TherapistId']?>&patientId=<?=$patientId?>">See DVRs</a>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                                <a class="btn btn-info" target="_blank" href="Allchargeslist.php?therapistId=<?=$row_refer['TherapistId']?>&PatientId=<?=$patientId?>">See Charges</a>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                                <a class="btn btn-info" href="javascript:void(0);" onclick="makeSms_popup('<?php echo '+1'.str_replace(array('(',')','-',' '), '', $therapist_phone); ?>','<?php echo $therapist_name;?>');" title="SMS">Send TEXT</a>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                                <button class="btn btn-info" id="faxattachment">See Fax Document(s)</button>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                                <button id="fax-attachment" class="btn btn-info" data-toggle="modal" data-target="#faxAttachmentModal">Upload Fax Document(s)</button><br><br>
   <?php //if(count($cert_periods) > 1){ ?>
<!--                                  <div class="alert alert-warning custom-alert pull-left" style="width:100%; top:-20px;">                                      
                                      <div class="row">
                                          <div class="col-sm-3"><label>Old Certification Periods</label></div>
                                          <?php foreach ($cert_periods as $cert_period){ ?>
                                          <div class="col-sm-3">
                                              <a href="javascript:void(0);" onclick="show_notes_by_cert_period('<?php echo $cert_period->id; ?>','<?php echo $cert_period->patient_id; ?>','<?php echo $cert_period->cert_start; ?>','<?php echo $cert_period->cert_end; ?>')">
                                                  <?php echo date('m/d/Y',  strtotime($cert_period->cert_start))." To ".date('m/d/Y',  strtotime($cert_period->cert_end)); ?>
                                              </a>
                                          </div>
                                          <?php } ?>
                                      </div>
                                  </div>-->
                                  <?php //}
                                  
                                      if($hold_message=='yes'){
                                  ?>
                                      <div class="alert alert-info custom-alert pull-left" style="width:100%; margin-top: 0 !important;">
                                          Patient is currently on Hold status. To resume services you must submit SPOF form w/ freq 1W1 and effective date same as Re-Eval date. After that you must complete Re-Eval, POC and Clin Note forms (and Tinetti if applicable)
                                      </div>
                                  <?php
                                      }
                                  ?>
                                 
                                    
                                
                                
                                  <div class="" style="width:100%">
                                      <?php // if($freq_note!=''){ echo $freq_note;} ?>
                                      <?php if ($pvt_insu_data) { ?>
                                      <table class="table table-stripped text-center" style="border: 1px solid #ddd; margin-bottom: 20px;">
                                          <tr style="background-color:#f3f4f5;">
                                                  <td><b>Entry #</b></td>
                                                  <td><b>Entered On</b></td>
                                                  <td><b>Visit Authorized</b></td>
                                                  <td><b>Valid dates</b></td>
                                                  <td><b>Visits made</b></td>
                                                  <td><b>Status</b></td>
                                              </tr>
                                      <?php 
                                            $pvt_count = 0;
                                            #echo '<pre>'; print_r($pvt_insu_data); exit;
                                            foreach($pvt_insu_data as $pvt_insu) { 
                                              $pvt_count++;
                                              $insu_status = $pvt_insu->status;
                                              if($pvt_insu->visit_details!=""){
                                                $count_total_visits = count(explode(',', $pvt_insu->visit_details));
                                                if($insu_status=='Continue' && $count_total_visits==$pvt_insu->visit_authorized){
                                                  //UPdate
                                                  $curAgency->updateData('tbl_private_insurence',array('status'=>"Completed"),"id = ".$pvt_insu->id);
                                                  $insu_status = "Completed";
                                                }
                                              }
                                              
                                      ?>
                                          <tr>
                                                  <td><?php echo $pvt_count.'.'; ?></td>
                                                  <td><?php if($pvt_insu->created_date !='' && strtotime($pvt_insu->created_date)){ echo date('m/d/Y',  strtotime($pvt_insu->created_date)); }else { echo '-'; } ?></td>
                                                  <td><?php echo $pvt_insu->visit_authorized; ?></td>
                                                  <td><?php echo date("m/d/Y", strtotime(str_replace("-", "/", $pvt_insu->from_date))); ?> to 
                                                  <?php echo date("m/d/Y", strtotime(str_replace("-", "/", $pvt_insu->to_date))); ?></td>
                                                  <td><?php echo $pvt_insu->visit_details; ?></td>
                                                  <td>
                                                      <?php if($insu_status =='Continue'){ echo '<span class="badge bg-green"><b>Active</b></span>'; }
                                                            else if($insu_status =='Completed'){ echo '<span class="badge bg-light-blue"><b>Completed</b></span>'; }
                                                            else{ echo '<span class="badge bg-red"><b>Stopped</b></span>'; }?>
                                                  </td>
                                              </tr>
                                      <?php } ?> </table> <?php } ?>                           
                                      
                                  </div>
                                  
                                  <div style="float:left; width:100%;">                                        
                                      <?php
                                       if($total_auth_visit_count >= $total_auth_visit && $total_auth_visit != 0){ ?>
                                           <!--<div style="font-size:14px; margin-bottom:10px;"><span style="color:#FF0000;">The <?=$total_auth_visit?> number of Authorized Visits has been reached</span></div>-->
                                          <div style="background-color: rgb(255, 0, 0); margin-bottom:10px; color: rgb(255, 255, 255); display: block; padding-left: 5px; font-size: 20px;"><span style="color:#FFFFFF;">The <?=$total_auth_visit?> number of Authorized Visits has been reached</span></div>
                                       <?php } ?>
                                  </div>
                                </div>

                                <div id="loadingEffect" class="notification information png_bg">
                                    <div>
                                        <img src="resources/images/ajax-loader.gif" /> Processing request, please wait...
                                    </div>
                                </div>
                                
                                
                                  <div id="accordion" class="box-group">
                                        
                                    
                                    
<!--                                    /**********************************************************/-->
                                <div class="demo">
                                      <div class="panel-group patient-forms-accordion" id="accordion" role="tablist" aria-multiselectable="true">
                                        <?php
                                       // added for episode samrat (06-05-2018)
                                       $count =0;
                                       $episodenumber = count($cert_periods);
                                       $tcount = count($cert_periods);
                                       $notsubmittedcount =0;
                                       if (count($cert_periods) > 0) {?>
                                      <?php foreach ($cert_periods as $cert_period) {
                                        $count++;
                                       $notsubmittedcount++;
                                      // added condition samrat (06-05-2018)

                                      $fromDate = strtotime($cert_period->cert_start);
                                      $toDate = strtotime($cert_period->cert_end);
                                        ?>
                                        <div class="panel panel-default" style="margin-bottom:5px;">
                                         <div class="panel-heading" role="tab" id="heading<?=$count?>">
                                          <h4 class="panel-title">
                                              <a role="button" data-toggle="collapse" data-parent="#accordion" href="#<?php echo $count; ?>collapseOne" aria-expanded="true" aria-controls="collapseOne">
                                                  Episode #<?php echo $episodenumber; ?> -(Start date: <?php echo date('m/d/Y',  strtotime($cert_period->cert_start)) ?> - End date: <?php echo date('m/d/Y',  strtotime($cert_period->cert_end)) ?>)
                                              </a>
                                          </h4>
                                         </div> 
                                            <?php
                                            if($count == 1){
                                              ?>
                                          <div id="<?php echo $count; ?>collapseOne" class="panel-collapse collapse in" role="tabpanel" aria-labelledby="heading<?=$count?>">
                                            <?php
                                            }else{
                                              ?>
                                          <div id="<?php echo $count; ?>collapseOne" class="panel-collapse collapse" role="tabpanel" aria-labelledby="heading<?=$count?>">
                                            <?php
                                            }
                                            ?>
                                          <div class="sub-accordion">
                                              <style>
                                              .expand-arrow a { border: 1px solid #ccc; padding: 5px; margin-bottom: 10px; display: inline-block;}
                                          </style>
                                          <div class="expand-arrow">
                                              <a href="javascript:void(0);" id="expandall<?=$count?>" title="Expand All"><img src="images/arrow-up.gif" /></a>
                                              <a href="javascript:void(0);" id="collapseall<?=$count?>" style="display:none;" title="Collapse All"><img src="images/arrow-dn.gif"/></a>
                                          </div>
                                          <div class="panel-group wrap" id="bs-collapse<?=$count?>">
                                    <!-- div class="" id="tab1"> <!-- This is the target div. id must match the href of this div's tab -->          
                                        <?php
                                        $curForm = new CForms();
                                        $result = $curForm->GetFormList();
                                        $evalCtr = 0;

                                        while ($row = $result->fetch_assoc()) {

                                        #echo "<pre>"; print_r($row);
                                        $formId = $row["Id"];
                                        if ($formId == 6) {
                                        $ctr = GetFormCountmissed($row["TableName"],$fromDate,$toDate);
                                        } else {
                                        $ctr = GetFormCount($row["TableName"],$fromDate,$toDate,$count,$tcount);
                                        }
                                        if ($row["Priority"] == 1) {
                                        $evalCtr = $ctr;
                                        }

                                        ?>
                                        <div class="panel">
                                          <div class="panel-heading">
                                              <h4 class="panel-title">
                                                    <a href="#<?=$count.''.$formId?>" data-parent="#bs-collapse<?=$count?>" data-toggle="collapse" class="collapsed">
                                                        <?=$row["Priority"]?>.
                                        <?php

                                               /* echo "<div class='panelPatientFormsCollapsed' >\n";
                                                echo "<h2>" . $row["Priority"] . ".";
                                                echo "<span style='color:#A83E20;font-weight:bold;'>";
                                                echo "&nbsp;&nbsp;&nbsp;";
                                                echo "</span>";*/

                                                if($formId == 3)
                                                {
                                                    echo $row["shortTitle"].' ('.$ctr.') Forms </a>';
                                                    echo "<div class='new-blank-form'><div style='float: right;'>";
                                                            //if($row_refer['activeTransferFlag'] && $evalCtr>0)
//                                                    if($count == 1){
                                                        echo "<button style='float:right;' class='btn btn-info' href='javascript:void(0)' url='" . $row["Url"] . ".php?referralId=" . $referralId . "'' onclick='check_visit_status(".$formId.", this )'>New blank form</button>";
//                                                    }
                                                    echo "</div>";
                                                    echo "<div style='float:right; margin:6px 23px 0;color:red;'><strong>Visits Remaining in Current POC : $total_visit_remaining</strong></div></div>\n";

                                                }



                                                else if($formId == 5)
                                                { 
                                                   
                                                    echo $row["shortTitle"].' ('.$ctr.') Forms </a>';
                                                        //echo "</a>\n";  
                                                    #$due_dt = date('m/d/Y',strtotime("+29day ".$TI_1));              
                                                    if($totalpoma < 2 && $totalpoma > 0) 
                                                    { 
                                                        if(($_SESSION["userType"] == 1 || $_SESSION["userType"] == 4 || $_SESSION['userType'] == 0))
                                                        {
                                                            //if ($row["Priority"]==1 || $evalCtr==1)

                                                        echo "<div class='new-blank-form'><div onclick='tinetti2();'>";
                                                           // if($row_refer['activeTransferFlag']) 
                                                        echo "<button style='float:right;' href='#' class='btn btn-info'>New blank form</button>";
                                                        echo "<div style='float:right; margin: 9px 25px 0 0; color:red;'><strong>Due On : $due_dt</strong></div></div>\n";
                                                        } 

                                                    }
                                                    else
                                                    {
                                                        if($_SESSION["userType"] == 1 || $_SESSION["userType"] == 4 || $_SESSION['userType'] == 0)
                                                        {
//                                                            if ($row["Priority"]==1 || $evalCtr>0)
//                                                            {
                                                                echo "<div class='new-blank-form'>";  
                                                                if($row_refer['activeTransferFlag'] )
                                                                echo "<button style='float:right;' href='javascript:void(0)' url ='" . $row["Url"] . ".php?referralId=" . $referralId . "&date=".date("Y-m-d", strtotime($last_reassessment))."' class='btn btn-info' onclick='check_visit_status(".$formId.", this)'>New blank form</button>";
                                                                if(strtotime($due_dt)){
                                                                echo "<div style='margin: 9px 25px 0 0; float: right;'><strong>Due On : $due_dt</strong></div></div>\n";
                                                                }


                                                            //}
                                                        }           
                                                    }             
                                                }
                                                else if($formId == '6')
                                                {


                                                    //echo "&nbsp;<a href='" . $row["Url"] . ".php?referralId=" . $referralId . "' style='text-decoration:underline;'  >";
                                                    echo $row["shortTitle"].' ('.$ctr.') Forms </a>';
                                                    //echo "</a>\n";
                                                    //if($_SESSION["userType"] == 1)
                                                    //{
                                                    echo "<div class='new-blank-form'>";
                                                    if($row_refer['activeTransferFlag'] )
                                                                echo "<button style='float:right;' href='javascript:void(0)' url='" . $row["Url"] . ".php?type=new&referralId=" . $referralId . "'' class='btn btn-info' onclick='check_visit_status(".$formId.", this)'>New blank form</button>";
                                                                echo "</div>\n";
                                                   // }


                                                }
                                                else if($formId == '8')
                                                {
                                                    //echo "&nbsp;<a href='" . $row["Url"] . ".php?referralId=" . $referralId . "' style='text-decoration:underline;'  >";
                                                    echo $row["shortTitle"].' ('.$ctr.') Forms </a>';
                                                    //echo "</a>\n";
                                                    if($_SESSION["userType"] == 1 || $_SESSION["userType"] == 4 || $_SESSION['userType'] == 0)
                                                    {
                                                    echo "<div class='new-blank-form'>";
                                                    if($row_refer['activeTransferFlag'] )
                                                                echo "<button style='float:right;' href='javascript:void(0)' url='" . $row["Url"] . ".php?referralId=" . $referralId . "'' class='btn btn-info' onclick='check_visit_status(".$formId.", this)'>New blank form</button>";
                                                                echo "</div>\n";
                                                    }
                                                }
                                                else
                                                {             
                                                        //echo "&nbsp;<a href='" . $row["Url"] . ".php?referralId=" . $referralId . "' style='text-decoration:underline;'  >";
                                                    if($formId==7 || $formId==9)
                                                    {
                                                      
                                                       if($row["shortTitle"]=='CC Note' && $ctr=='0'){
                                                        echo "<strong style='color:red;border: 1px solid red;'>".$row["shortTitle"]."(0)Forms</strong>";
                                                       }
                                                       if($row["shortTitle"]=='SPOF' && $ctr=='0'){
                                                        echo $row["shortTitle"].' ('.$ctr.') Forms </a>';
                                                      }
                                                        
                                                        if(($_SESSION["userType"] == 1 || $_SESSION["userType"] == 4 || $_SESSION['userType'] == 0) )
                                                        //echo "<div style='float:right;margin-right:300px;'>";
                                                        //if($row_refer['activeTransferFlag'])
                                                                echo "<div class='new-blank-form'><button style='float:right;' href='javascript:void(0)' url='" . $row["Url"] . ".php?referralId=" . $referralId . "'' class='btn btn-info' onclick='check_visit_status(".$formId.", this)'>New blank form</button></div>";
                                                                //echo "</div>\n";
                                                    }
                                                    else
                                                    { 
                                                            //if ($row["Priority"]==1 || $evalCtr>0)
                                                            //{
                                                                echo $row["shortTitle"].' ('.$ctr.') Forms </a>';
                                                                if($_SESSION["userType"] == 1 || $_SESSION["userType"] == 4 || $_SESSION['userType'] == 0)
                                                                { 
                                                                if($formId!=3){             
                                                                //echo "</a>\n";
                                                                //echo "<div style='float:right;margin-right:300px;'>";
//                                                                if($count == 1){
                                                                    if($formId==4){
                                                                        echo "<div class='new-blank-form'><button style='float:right;' href='javascript:void(0)' url='" . $row["Url"] . ".php?referralId=" . $referralId . "''  class='btn btn-info' onclick='check_visit_status(".$formId.", this)'>New blank form</button></div>";  
                                                                    }else{
                                                                        echo "<div class='new-blank-form'><button style='float:right;' href='javascript:void(0)' url='" . $row["Url"] . ".php?referralId=" . $referralId . "'' class='btn btn-info' onclick='check_visit_status(".$formId.", this)'>New blank form</button></div>";   
                                                                    }

//                                                                }

                                                                //echo "</div>\n";
                                                                //}
                                                                }
                                                            }
                                                             else {           
                                                                echo $row["shortTitle"].' Forms';
                                                            }
                                                    }

                                                }

                                                /*if($formId == '5')
                                                {
                                                    print "<div style='float:right;'>Due On : 31-1-2012</div>";
                                                }*/
                                                ?>
                                                        </h4>
                                            </div>
                                        
                                            <?php 
                                            if($formId == 1){
                                            ?>
                                            <div class="panel-collapse collapse in expand<?=$count?>" id="<?=$count.''.$formId?>">
                                            <?php
                                            }else{
                                              ?>
                                            <div class="panel-collapse collapse expand<?=$count?>" id="<?=$count.''.$formId?>">
                                            <?php
                                            }
                                            ?>
                                                <div class="panel-body">
                                                    
                                               
                                        <?php
                                       // echo "<div class='panelcontent'>";
                                        DisplaySummary($row["Id"],$fromDate,$toDate,$tcount,$notsubmittedcount,$count);
                                       // echo "</div>\n";
                                        //echo "</div>\n";
                                        ?>
                                                     </div>
                                            </div>
                                    </div>
                                         <?php
                                        }


                                        $result->close();
                                        ?>
                                                    
                                        <!--START OASIS FORM CODE -->
                                        <?php if ($oasis_status == "checked") { ?>
                                          <div class="panel">
                                                <div class="panel-heading">
                                                <h4 class="panel-title">
                                                    <a href="#<?=$count?>collapseThree" data-parent="#bs-collapse<?=$count?>" data-toggle="collapse" class="collapsed">
                                                        10. D/C OASIS (<?php 
                                                        $w1                           = " visit_date BETWEEN ".date("'Y-m-d'",$fromDate)." AND ".date("'Y-m-d'",$toDate)." AND referralId IN (" . $setallflagfalse['ids'] . ") AND form_status = 'submitted'";
                                                        $count_oasis_list1            = $curAgency->selectData('tbl_oasis', array('id'), $w1);
                                                        $count_oasis_list_newversion1 = $curAgency->selectData('tbl_oasis_newversion', array('id'), $w1);
                                                        echo count($count_oasis_list1) + count($count_oasis_list_newversion1); ?>) Forms
                                                    </a>
                                                    <div class="new-blank-form">
                                          <?php if ($row_refer['activeTransferFlag'] && ($_SESSION['userType'] == 3 || $_SESSION['userType'] == 1 || $_SESSION["userType"] == 4 || $_SESSION['userType'] == 0) ) { ?>
                                                        <!--<a class="btn btn-info" onclick="check_visit_status(10, this);" href='javascript:void(0)' url="oasis_update.php?type=new&referralId=< ?= $_REQUEST['referralId'] ?>">New blank form</a>< ?php } ?>-->
                                                         <button class="btn btn-info" style='float:right;' onclick="check_visit_status(10, this);" href='javascript:void(0)' url="oasis_newversion.php?type=new&referralId=<?= $_REQUEST['referralId'] ?>">New blank form</button><?php } ?>
                                                      </div>
                                                </h4>
                                            </div>
                                              
                                             <div class="panel-collapse collapse expand<?=$count?>" id="<?=$count?>collapseThree">
                                                <div class="panel-body">

                                                  <table width="100%" class="sub-accordion-table"><thead><tr style=""><th>&nbsp;</th><th>Date Created</th><th>Date on Form</th><th>Submitted?</th></tr> </thead><tbody>

                                          <?php
                                          $cnt = 0;
                                         if (count($oasis_list) > 0 || count($oasis_list_new) > 0) {
                                            
                                            foreach ($oasis_list_new as $item) {
                                                if((strtotime($item->visit_date) >= strtotime($cert_period->cert_start) && strtotime($item->visit_date) <= strtotime($cert_period->cert_end)) || ($item->visit_date == '0000-00-00' && $notsubmittedcount==1)){
                                                $oasis_update = "select update_version from tbl_oasis_newversion where referralId=" . $_SESSION['referralId'] . " and id=" . $item->id . " ";
                                              
                                              $result_update_oasis = $curAgency->GetDataList($oasis_update);
                                              if ($result_update_oasis)
                                                $update_oasisdata = $result_update_oasis->fetch_assoc();
                                              ($update_oasisdata['update_version']);

                                              #echo "<pre>"; print_r($item);
                                              $cnt++;
                                                if($item->form_status == "saved" && $row_refer['dischargeFlag'] == 0 && $row_refer['deleteFlag'] == 0)
                                                                {
                                                                    $tr_style = 'style="background-color:#ebccd1"';
                                                                }
                                                                else
                                                                {
                                                                    $tr_style = '';
                                                                }
                                                ?>
                                                                        <tr <?=$tr_style?>>

                                                                      <td><?= $cnt ?></td>

                                              <?php if (($update_oasisdata['update_version']) != 0) { ?>
                                                                        <td>&nbsp;<a href="oasis_newversion.php?type=edit&referralId=<?= $item->referralId ?>&id=<?= $item->id ?>" class="aLink"><?= (strtotime($item->created_date) && $item->created_date != '0000-00-00') ? date("m/d/Y", strtotime($item->created_date)) : '' ?></a></td>
                                              <?php } else { ?>
                                                                        <td>&nbsp;<a href="oasis.php?type=edit&referralId=<?= $item->referralId ?>&id=<?= $item->id ?>" class="aLink"><?= (strtotime($item->created_date) && $item->created_date != '0000-00-00') ? date("m/d/Y", strtotime($item->created_date)) : '' ?></a></td>
                                              <?php } ?>
                                                                      <td>&nbsp;<?= (strtotime($item->visit_date) && $item->visit_date != '0000-00-00') ? date("m/d/Y", strtotime($item->visit_date)) : '' ?> </td>
                                                                      <?php if($item->form_status == "saved"){$submit_check = 1;}?>
                                                                      <td>&nbsp;<?= ($item->form_status == "saved") ? 'No' : 'Yes' ?><?= ($item->cancel_flag == "yes") ? '<span style="color:red">&nbsp;Cancelled</span>' : '' ?>&nbsp;</td>
                                                                  </tr>
                                            <?php }
                                            }
                                            foreach ($oasis_list as $item) {
                                                if((strtotime($item->visit_date) >= strtotime($cert_period->cert_start) && strtotime($item->visit_date) <= strtotime($cert_period->cert_end)) || ($item->visit_date == '0000-00-00' && $notsubmittedcount == 1)){
                                                $oasis_update = "select update_version from tbl_oasis where referralId=" . $_SESSION['referralId'] . " and id=" . $item->id . " ";
                                              
                                              $result_update_oasis = $curAgency->GetDataList($oasis_update);
                                              if ($result_update_oasis)
                                                $update_oasisdata = $result_update_oasis->fetch_assoc();
                                              ($update_oasisdata['update_version']);

                                              #echo "<pre>"; print_r($item);
                                              $cnt++;

                                              if($item->form_status == "saved" && $row_refer['dischargeFlag'] == 0 && $row_refer['deleteFlag'] == 0){
                                               $tr_style = "style='background-color:#ebccd1'";
                                              }
                                              else{                                              
                                                 $tr_style = '';
                                              }
                                             ?>
                                                                        <tr <?=$tr_style?>>

                                                                      <td><?= $cnt ?></td>

                                              <?php if (($update_oasisdata['update_version']) != 0) { ?>
                                                                        <td>&nbsp;<a href="oasis_update.php?type=edit&referralId=<?= $item->referralId ?>&id=<?= $item->id ?>" class="aLink"><?= (strtotime($item->created_date) && $item->created_date != '0000-00-00') ? date("m/d/Y", strtotime($item->created_date)) : '' ?></a></td>
                                              <?php } else { ?>
                                                                        <td>&nbsp;<a href="oasis.php?type=edit&referralId=<?= $item->referralId ?>&id=<?= $item->id ?>" class="aLink"><?= (strtotime($item->created_date) && $item->created_date != '0000-00-00') ? date("m/d/Y", strtotime($item->created_date)) : '' ?></a></td>
                                              <?php } ?>
                                                                      <td>&nbsp;<?= (strtotime($item->visit_date) && $item->visit_date != '0000-00-00') ? date("m/d/Y", strtotime($item->visit_date)) : '' ?> </td>
                                                                      <?php if($item->form_status == "saved"){$submit_check = 1;}?>
                                                                      <td>&nbsp;<?= ($item->form_status == "saved") ? 'No' : 'Yes' ?><?= ($item->cancel_flag == "yes") ? '<span style="color:red">&nbsp;Cancelled</span>' : '' ?>&nbsp;</td>
                                                                  </tr>
                                            <?php }
                                            }
                                          } else { ?>
                                                                    
                                                                <tr>
                                                                    <td colspan="5">No form(s) submitted yet.</td>
                                                                </tr>     
                                            <?php
                                          }
                                          ?>

                                                      </tbody></table>

                                              </div>

                                          </div>
                                              </div>
                                        <?php } ?>
                                        <!--END OASIS FORM CODE-->
                                            </div>
                                          </div>
                                          </div>
                                        </div>  
                                  
                                
                            
                                          <script type="text/javascript">   
          //expand all and collapse all funtionality by samrat(09/07/2018)
      $('#expandall<?=$count?>').click(function(){
        $('.expand<?=$count?>').addClass('in');
        $('.in').css({'height':'auto'});
        $('#expandall<?=$count?>').hide();
        $('#collapseall<?=$count?>').show();
      });

      $('#collapseall<?=$count?>').click(function(){
        $('.expand<?=$count?>').removeClass('in');
        $('.expand<?=$count?>').css({'height':'0'});
        $('#collapseall<?=$count?>').hide();
        $('#expandall<?=$count?>').show();
      });

            function toggleIcon(e) {
                  $(e.target)
                      .prev('.panel-heading')
                      .find(".more-less")
                      .toggleClass('glyphicon-plus glyphicon-minus');
              }
              $('.panel-group').on('hidden.bs.collapse', toggleIcon);
              $('.panel-group').on('shown.bs.collapse', toggleIcon);


               $(document).ready(function () {
                          $('.collapse.in').prev('.panel-heading').addClass('active');
                          $('#accordion, #bs-collapse<?=$count?>')
                              .on('show.bs.collapse', function (a) {
                                  $(a.target).prev('.panel-heading').addClass('active');
                              })
                              .on('hide.bs.collapse', function (a) {
                                  $(a.target).prev('.panel-heading').removeClass('active');
                              });
                      });
      </script>
                                           <?php 
                                    $episodenumber--;
                                       }
                                    }
                                    ?>  
                                    </div>
                                     </div> 
                            </div>
                            </div>
                        </div>
                            
                        </div>
                    
                </div>
                <!-- InstanceEndEditable -->
                                <div class="footer">
                                                    <?php require_once("scripts/footer.php") ?>
                                </div>
            </section>
        </aside>
    </body>
    <!-- InstanceEnd --></html>
<style type="text/css">
  .sel_bc_color{
    background-color:hsl(350deg 44% 86%);
  }
</style>
      <?php

      function GetFormCount($tblName,$fromDate,$toDate,$count,$tcount) {
        $ctr = 0;
        global $referralId;
        global $row_refer;
        global $patientId;
        $curResponse = new CResponses();
        $curAgency = new CAgencies();
        $sql = "select count(*) as ctr from " . $tblName;
        // start added by samrat (07-06-2018) for count 
    if($tblName == 'tblEval'){
      $sql .= " WHERE TI_1 BETWEEN ".date("'Y-m-d'",$fromDate)." AND ".date("'Y-m-d'",$toDate);
    }elseif($tblName == 'tblPOC'){
      $sql .= " WHERE STR_TO_DATE(PTPOFEVALVI_1, '%m-%d-%Y') BETWEEN ".date("'Y-m-d'",$fromDate)." AND ".date("'Y-m-d'",$toDate);
    }elseif($tblName == 'tblNote'){
       $sql .= " WHERE DATE_FORMAT(STR_TO_DATE(TI_4, '%m/%d/%Y'), '%Y-%m-%d') BETWEEN ".date("'Y-m-d'",$fromDate)." AND ".date("'Y-m-d'",$toDate);
    }elseif($tblName == 'tbldischarge'){
       $sql .= " WHERE DATE_FORMAT(STR_TO_DATE(PI_2, '%m/%d/%Y'), '%Y-%m-%d') BETWEEN ".date("'Y-m-d'",$fromDate)." AND ".date("'Y-m-d'",$toDate);
    }elseif($tblName == 'tbl30day'){
       $sql .= " WHERE DATE_FORMAT(STR_TO_DATE(cVisitDate, '%m/%d/%Y'), '%Y-%m-%d') BETWEEN ".date("'Y-m-d'",$fromDate)." AND ".date("'Y-m-d'",$toDate);
    }elseif($tblName == 'tblSPOF'){
       $sql .= " WHERE DATE_FORMAT(STR_TO_DATE(PO_29, '%m/%d/%Y'), '%Y-%m-%d') BETWEEN ".date("'Y-m-d'",$fromDate)." AND ".date("'Y-m-d'",$toDate);
    }elseif($tblName == 'tblPOMA'){
       $sql .= " WHERE PT_1 BETWEEN ".date("'Y-m-d'",$fromDate)." AND ".date("'Y-m-d'",$toDate);
    }elseif($tblName == 'tblCCNote'){
        if($count == $tcount){
       $sql .= " WHERE DATE_FORMAT(STR_TO_DATE(PO_5, '%m/%d/%Y'), '%Y-%m-%d') <= ".date("'Y-m-d'",$toDate);
      }else{
       $sql .= " WHERE DATE_FORMAT(STR_TO_DATE(PO_5, '%m/%d/%Y'), '%Y-%m-%d') BETWEEN ".date("'Y-m-d'",$fromDate)." AND ".date("'Y-m-d'",$toDate);
      }
    }else{
       $sql .= " WHERE StatusId=2";
    }
      //end added by samrat (07-06-2018) for count 
     $sql .= " AND StatusId=2 AND  ReferralId ";
        if (!$row_refer['dischargeFlag']) {
          $refer_id = "SELECT group_concat(r.Id) as ids FROM tblreferrals r JOIN tblusers u WHERE r.TherapistId = u.TherapistId AND u.UserType = 1 AND r.dischargeFlag=" . $row_refer[dischargeFlag] . " AND r.PatientId = " . $patientId . "";

          $result_find_referral = $curAgency->GetDataList($refer_id);
          if ($result_find_referral)
            $refer_ids = $result_find_referral->fetch_assoc();
          if ($refer_ids['ids'] == "")
            $refer_ids['ids'] = "''";

          #echo "<pre>"; print_r($refer_ids);

          $sql .= " IN(" . $refer_ids['ids'] . ")";
        }
        else {
          //$sql .=" = ".$referralId;
          #$refer_id = "SELECT group_concat(r.Id) as ids FROM tblreferrals r JOIN tblusers u WHERE r.TherapistId = u.TherapistId AND u.UserType = 1 AND r.dischargeFlag= 1 AND r.PatientId = " . $row_refer[PatientId] . " AND r.AgencyId=" . $row_refer[AgencyId] . " AND r.dischargeDate = '" . $row_refer[dischargeDate] . "'";
          $refer_id = "SELECT group_concat(r.Id) as ids FROM tblreferrals r JOIN tblusers u WHERE r.TherapistId = u.TherapistId AND u.UserType = 1 AND r.PatientId = " . $row_refer[PatientId] . " AND r.AgencyId=" . $row_refer[AgencyId];
          $result_find_referral = $curAgency->GetDataList($refer_id);
          if ($result_find_referral)
            $refer_ids = $result_find_referral->fetch_assoc();

          if ($refer_ids['ids'] == "")
            $refer_ids['ids'] = "''";
          $sql .= " IN(" . $refer_ids['ids'] . ")";
        }


        //. $referralId;
        //$sql .= " where StatusId=2 AND ReferralId=" . $referralId;
        //$sql .= " and StatusId=1"; //submitted.
        //  echo $sql;
        $result = $curResponse->ExecuteSql($sql);
        $row = $result->fetch_assoc();
        $ctr = $row["ctr"];
        #echo $sql;
        $result->close();
        return $ctr;
      }

      function GetFormCountmissed($tblName,$fromDate,$toDate) {
        $ctr = 0;
        global $referralId;
        global $row_refer;
        global $patientId;
        $curResponse = new CResponses();
        $curAgency = new CAgencies();
        $sql = "select count(*) as ctr from " . $tblName;
        if($tblName == 'tblmissedvisit'){
       $sql .= " WHERE visit_date BETWEEN ".date("'Y-m-d'",$fromDate)." AND ".date("'Y-m-d'",$toDate);
    }else{
       $sql .= " WHERE form_status='submitted'";
    }
    $sql .= " AND form_status='submitted' AND referralId ";
    
        if (!$row_refer['dischargeFlag']) {
          $refer_id = "SELECT group_concat(r.Id) as ids FROM tblreferrals r JOIN tblusers u WHERE r.TherapistId = u.TherapistId AND u.UserType = 1 AND r.StatusId = 2 AND r.dischargeFlag=" . $row_refer[dischargeFlag] . " AND r.PatientId = " . $patientId . "";

          $result_find_referral = $curAgency->GetDataList($refer_id);

          if ($result_find_referral)
            $refer_ids = $result_find_referral->fetch_assoc();
          if ($refer_ids['ids'] == "")
            $refer_ids['ids'] = "''";

          $sql .= " IN(" . $refer_ids['ids'] . ")";
        }
        else {
          $refer_id = "SELECT group_concat(r.Id) as ids FROM tblreferrals r JOIN tblusers u WHERE r.TherapistId = u.TherapistId AND u.UserType = 1 AND r.StatusId = 2 AND r.dischargeFlag= 1 AND r.PatientId = " . $row_refer[PatientId] . " AND r.AgencyId=" . $row_refer[AgencyId] . " AND r.dischargeDate = '" . $row_refer[dischargeDate] . "'";

          $result_find_referral = $curAgency->GetDataList($refer_id);
          if ($result_find_referral)
            $refer_ids = $result_find_referral->fetch_assoc();
          if ($refer_ids['ids'] == "")
            $refer_ids['ids'] = "''";

          $sql .= " IN(" . $refer_ids['ids'] . ")";
        }

        //. $referralId;
        //$sql .= " where StatusId=2 AND ReferralId=" . $referralId;
        //$sql .= " and StatusId=1"; //submitted.
        //  echo $sql;
        $result = $curResponse->ExecuteSql($sql);
        $row = $result->fetch_assoc();
        $ctr = $row["ctr"];
        //echo $sql;
        $result->close();
        return $ctr;
      }

      function DisplaySummary($formId,$fromDate,$toDate,$tcount,$notsubmittedcount,$count) {


        global $referralId;
        global $TI_1;
        global $reassment_dt;
        global $patientId;
        global $row_refer;
        #echo "<pre>"; print_r($_SESSION);
        //echo $referralId;
        //echo $patientId;
        $reassment_dt1 = "";
        $curForm = new CForms();
        $curAgency = new CAgencies();
        $result = $curForm->GetFormSummaryByRefId_nelson($referralId, $formId, $patientId);
        if($_SERVER['REMOTE_ADDR']=='111.93.228.190' && $formId ==9){
          
//        echo "<pre>"; print_r($result);
        }

        echo '<table class="form-th" cellpadding="0" cellspacing="0" style="width:100%;">';
        echo "<thead>";
        echo "<tr >";
        echo "<th>&nbsp;</th>";
        echo "<th>Date Created</th>";
        echo "<th>Date on Form</th>";
        
        if ($formId == 2)
          echo "<th>Frequency</th>";
        if($formId == 6 || $formId == 7)
          echo "<th class='text-center'>New Frequency</th><th class='text-center'>Effective Date</th>";
        
        echo "<th>Submitted?</th>";        
        echo "</tr> ";
        echo "</thead>";
        echo "<tbody>";
        $rowStyle = "";
        $totalRows = 0;

        $link = GetFormLink($formId);
        if ($formId == 6)
          $link .= "&referralId=";
        else
          $link .= "?referralId=";

        $rowLink = "";
        $mouseOver = "this.style.backgroundColor='#cfcfcf';this.style.borderColor='#cfcfcf'; this.style.cursor = 'pointer';this.style.cursor='hand';";
        $mouseOut = "this.style.backgroundColor='#FFFFFF';";
        $filter_ES_date = array();



        #$TI_1_arr = array();
        $visit_date_arr = array();
        $arr_cnt = 0;
        
        //$query = "SELECT group_concat(r.Id ORDER BY r.Id ASC) AS rId FROM tblreferrals r JOIN tblusers u WHERE r.TherapistId = u.TherapistId AND u.UserType = 1 AND r.PatientId = " . $_SESSION['selectedPatientId'] . " AND r.dischargeFlag != 1 AND r.deleteFlag = 0";
        $query = "SELECT group_concat(r.Id ORDER BY r.Id ASC) AS rId FROM tblreferrals r JOIN tblusers u WHERE r.TherapistId = u.TherapistId AND u.UserType = 1 AND r.PatientId = " . $_SESSION['selectedPatientId'] ;
        // echo $query;die;
        $find_referral = $curAgency->GetDataList($query);

        if ($find_referral)
          $resFind = $find_referral->fetch_assoc();
        #echo "<pre>"; print_r($resFind['rId']);
        $last_eval_date = $curAgency->selectData('tblEval', array('TI_1'), "ReferralId IN(" . $resFind['rId'] . ")", 'order by Id  DESC LIMIT 1');
        #echo "<pre>"; print_r($last_eval_date);

        while ($row = $result->fetch_array()) {
          
        
          if($count == $tcount){
//           $cccond = ((strtotime($row["PO_5"]) <= $toDate && $row["table_name"] == 'ccnote') || ($row["PO_5"] == '' && $notsubmittedcount==1 && $row["table_name"] == 'ccnote'));
           $cccond = (("DATE_FORMAT(STR_TO_DATE(PO_5, '%m/%d/%Y'), '%Y-%m-%d') <= ".date("'Y-m-d'",$toDate) && $row["table_name"] == 'ccnote') || ($row["PO_5"] == '' && $notsubmittedcount==1 && $row["table_name"] == 'ccnote'));
        }else{
//        $cccond = ((strtotime($row["PO_5"]) >= $fromDate && strtotime($row["PO_5"]) <= $toDate && $row["table_name"] == 'ccnote') || ($row["PO_5"] == '' && $notsubmittedcount==1 && $row["table_name"] == 'ccnote'));
        $cccond = ((("DATE_FORMAT(STR_TO_DATE(PO_5, '%m/%d/%Y'), '%Y-%m-%d') >= ".date("'Y-m-d'",$fromDate)) && ("DATE_FORMAT(STR_TO_DATE(PO_5, '%m/%d/%Y'), '%Y-%m-%d') <= ".date("'Y-m-d'",$toDate)) && $row["table_name"] == 'ccnote') || ($row["PO_5"] == '' && $notsubmittedcount==1 && $row["table_name"] == 'ccnote'));
       }
//         if($_SERVER['REMOTE_ADDR']=='111.93.228.190' && $formId == 9){
//          echo "<pre>"; print_r($row);   
//        }
      
     
 if(((strtotime($row["TI_1"]) >= $fromDate && strtotime($row["TI_1"]) <= $toDate && $row["table_name"] == 'eval') || ($row["TI_1"] == '0000-00-00' && $notsubmittedcount==1 && $row["table_name"] == 'eval'))
|| ((strtotime(str_replace("-","/",$row["PTPOFEVALVI_1"])) >= $fromDate && strtotime(str_replace("-","/",$row["PTPOFEVALVI_1"])) <= $toDate && $row["table_name"] == 'tpoc') || (str_replace("-","/",$row["PTPOFEVALVI_1"]) == '' && $notsubmittedcount==1 && $row["table_name"] == 'tpoc'))
|| ((strtotime(str_replace("-","/",$row["TI_4"])) >= $fromDate && strtotime(str_replace("-","/",($row["TI_4"]))) <= $toDate && $row["table_name"] == 'cnote') || ($row["TI_4"] == '' && $notsubmittedcount==1 && $row["table_name"] == 'cnote'))
|| ((strtotime(str_replace("-","/",$row["PI_2"])) >= $fromDate && strtotime(str_replace("-","/",$row["PI_2"])) <= $toDate && $row["table_name"] == 'tdischarge') || (str_replace("-","/",$row["PI_2"]) == '' && $notsubmittedcount==1 && $row["table_name"] == 'tdischarge'))
|| ((strtotime($row["cVisitDate"]) >= $fromDate && strtotime($row["cVisitDate"]) <= $toDate && $row["table_name"] == 'reassesment') || ($row["cVisitDate"] == '' && $notsubmittedcount==1)&& $row["table_name"] == 'reassesment')
|| ((strtotime($row["visit_date"]) >= $fromDate && strtotime($row["visit_date"]) <= $toDate && $row["table_name"] == 'missedvisit') || ($row["visit_date"] == '0000-00-00' && $notsubmittedcount==1 && $row["table_name"] == 'missedvisit'))
|| ((strtotime(date('Y-m-d', strtotime($row["PO_29"]))) >= $fromDate && strtotime(date('Y-m-d', strtotime($row["PO_29"]))) <= $toDate && $row["table_name"] == 'tspof') || ($row["PO_29"] == '' && $notsubmittedcount==1 && $row["table_name"] == 'tspof'))
// || ((("DATE_FORMAT(STR_TO_DATE(PO_29, '%m/%d/%Y'), '%Y-%m-%d') >= ".date("'Y-m-d'",$fromDate)) && ("DATE_FORMAT(STR_TO_DATE(PO_29, '%m/%d/%Y'), '%Y-%m-%d') <= ".date("'Y-m-d'",$toDate)) && $row["table_name"] == 'tspof') || ($row["PO_29"] == '' && $notsubmittedcount==1 && $row["table_name"] == 'tspof'))
|| ((strtotime($row["PT_1"]) >= $fromDate && strtotime($row["PT_1"]) <= $toDate && $row["table_name"] == 'tpoma') || ($row["PT_1"] == '' && $notsubmittedcount==1 && $row["table_name"] == 'tpoma'))
|| $cccond ) {

          $totalRows++;

          if ($formId == '5') {   #echo $GLOBALS['last_eval_Date'];
            $query = "select cVisitDate from tbl30day where Id = '$row[Id]'";
            
            $result1 = $curAgency->GetDataList($query);
            $ress_dt = $result1->fetch_Assoc();
            #echo "<pre>"; print_r($ress_dt); echo "</pre>";
            $visit_date_arr[$totalRows] = $ress_dt['cVisitDate'];
          }

          #$visit_date_arr[$totalRows] = $row['ES_Date'];
          if ($totalRows > 1) {

            $date = date("Y-m-d", strtotime($visit_date_arr[$totalRows - 1]));
            $dates = array($date, $last_eval_date[0]->TI_1);
            $most_recent = max($dates);
          }

          if ($totalRows == 1) {

            $query = "select cVisitDate from tbl30day where Id = '$row[Id]'";
            
            $result1 = $curAgency->GetDataList($query);
            if ($result1)
              $ress_dt = $result1->fetch_Assoc();
            $eval_date = $curAgency->selectData('tblEval', array('TI_1'), "ReferralId IN(" . $resFind['rId'] . ")", 'AND TI_1 < "' . date("Y-m-d", strtotime($ress_dt['cVisitDate'])) . '"order by Id  DESC LIMIT 1');
            // echo "<pre>"; print_r($eval_date);
            //echo date("Y-m-d", strtotime($ress_dt['cVisitDate']));
            //echo "hello";
            $most_recent = $eval_date[0]->TI_1;
          }


          if ($formId == '5') {
            #echo "<pre>"; print_r($visit_date_arr);
            $rowLink = $link . $row['ReferralId'] . "&Id=" . $row["Id"] . "&date=" . $most_recent;
          } else if ($formId == '6') {
            $rowLink = $link . $row['referralId'] . "&id=" . $row["id"];
          } else {
            $rowLink = $link . $row['ReferralId'] . "&Id=" . $row["Id"];
          }



          $status = "No";
          if ($formId == 6) {
            if ($row["form_status"] == "submitted")
              $status = "Yes";
          }
          else {
            if ($row["StatusId"] == 2)
              $status = "Yes"; 
          }
          $sel_cls = "";
          if($status == "No"){
            $sel_cls = "sel_bc_color";
          }

          

          $rowStyle = "  ";
          $rowStyle .= $sel_cls;
          if (($totalRows % 2) == 0)
            $rowStyle .= " sel-tr";
          //$onClick = "window.location='$rowLink';";
          
          $trstyle = '';
          if($row_refer['dischargeFlag'] == 0 && $row_refer['dischargeFlag'] == 0){
            if ($formId == 6) {
              if ($row["form_status"] == "saved")
                $trstyle = 'style="background-color:#ebccd1"';
            }
            else {
              if ($row["StatusId"] != 2 )
                $trstyle = 'style="background-color:#ebccd1"';
            }
          }
          
          
          // if ($status == "No"){
          // echo "<tr " . $rowStyle .  " " . $trstyle . " style='background-color:hsl(350deg 44% 86%)' >\n";
          // echo "<td style='background-color:hsl(350deg 44% 86%)'>\n";
          // }else{
            $rowStyle .= " class='". $rowStyle ."'";
          echo "<tr " . $rowStyle .  " " . $trstyle . " ' >\n";
          echo "<td>\n";
          // }
          echo $totalRows . ". ";
          /* if ($row["StatusId"]==1) {
            echo "<a href=\"" . $link . "\" title=\"Edit\"><img src=\"images/pencil.png\" alt=\"Edit\" border=\"0\" /></a>&nbsp;";
            echo "<a href=\"javascript:confirmDelete(" . $row["Id"] . ");\" title=\"Delete\"><img src=\"images/cross.png\" alt=\"Delete\" border=\"0\"  /></a> ";
            } */


          echo "</td>\n";
          echo "<td>&nbsp;<a href='$rowLink' class='aLink'>" . $row["DateCreated"] . "</a></td>\n";

          static $TI_1_arr;


          if ($formId == '1' || $formId == '2') {


            if (strtotime($row["TI_1"])) {
              $TI_1_arr[] = $row["TI_1"];

              //if($formId=='1')
              #$TI_1_arr[] = $TI_1 = $row["TI_1"];
              $TI_1 = $row["TI_1"];

              #echo "<pre>"; print_r($TI_1_arr); echo "</pre>";
              if ($totalRows == 1 && $formId == '1') {

                $reassment_dt = date('m/d/Y', strtotime($TI_1));
              }

              echo "<td>&nbsp;";
             if(!empty($TI_1) && $TI_1 != "0000-00-00")
                    echo date('m/d/Y', strtotime($TI_1));

              echo "</td>\n";
            } else
            if ($reassment_dt && $formId == '2') {
           $poc_date = $curAgency->selectData('tblEval', array('TI_1'), 'Id ='.$row["EvalId"]);
         echo "<td>&nbsp;"; 
         #echo $poc_date[0]->TI_1;
         if(strtotime($poc_date[0]->TI_1))
          echo date('m/d/Y',strtotime($poc_date[0]->TI_1));
          if (in_array(date('Y-m-d',strtotime($row['ES_Date'])), $filter_ES_date)) {
            echo " <span style='color:red'>Duplicate</span>";
          } 
          echo  "</td>\n";
        } else 
              echo "<td>&nbsp;</td>";
          }
          if ($formId == '5') {
            #echo "<pre>"; print_r($row);exit;
            ///$query = "select cVisitDate from tbl30day where Id = '$row[Id]'";
            if ($result1)
              $query = "select cVisitDate from tbl30day where Id = '$row[Id]'";
           
            $result1 = $curAgency->GetDataList($query);
            if ($result1)
              $ress_dt = $result1->fetch_Assoc();
            if (strtotime($ress_dt['cVisitDate'])) {
              //if($reassment_dt)
              echo "<td>&nbsp;" . $ress_dt['cVisitDate'];
              if (in_array($row['ES_Date'], $filter_ES_date)) {
                echo " <span style='color:red'>Duplicate</span>";
              }
              echo "</td>\n";
            } else {
              echo "<td>&nbsp;";
              if (in_array($row['ES_Date'], $filter_ES_date)) {
                echo " <span style='color:red'>Duplicate</span>";
              }
              echo "</td>";
            }
          }
          if ($formId == '3') {
            if (strtotime($row["TI_4"])) {
              echo "<td>&nbsp;" . date('m/d/Y', strtotime($row["TI_4"]));
              /*if (in_array($row['ES_Date'], $filter_ES_date)) {
                echo " <span style='color:red'>Duplicate</span>";
              }*/
              if (in_array($row['ES_Date'], $filter_ES_date) && ($row['cancel_old_form_id'] != '' && $row['cancel_old_form_id'] != 0)) {
                echo " <span style='color:red'>Duplicate</span>";
              }
              else if($row['cancel_old_form_id'] != '' && $row['cancel_old_form_id'] != 0){
                echo " <span style='color:red'>Duplicate</span>";
              }
              echo "</td>\n";
            } else {
              echo "<td>&nbsp;</td>";
            }
          }
          if ($formId == '4') {
            //print_r($row);exit;
            if (strtotime($row["PI_2"])) {
              echo "<td>&nbsp;" . date('m/d/Y', strtotime($row["PI_2"]));
              if (in_array($row['ES_Date'], $filter_ES_date)) {
                echo " <span style='color:red'>Duplicate</span>";
              }
              echo "</td>\n";
            } else {
              echo "<td>&nbsp;</td>";
            }
          }
          if ($formId == '6') {

            #echo  "<pre>";   print_r($row);exit;
            if (($row["visit_date"] != '0000-00-00')) {
              echo "<td>&nbsp;" . date('m/d/Y', strtotime($row["visit_date"]));

              $dt = explode(" ", $row["visit_date"]);
              #echo "<td>&nbsp;" . $dt[0];
              if (in_array($row['ES_Date'], $filter_ES_date)) {
                echo " <span style='color:red'>Duplicate</span>";
              }
              else if($row['cancel_old_form_id'] != '' && $row['cancel_old_form_id'] != 0){
                echo " <span style='color:red'>Duplicate</span>";
              }
              echo "</td>\n";
            } else {
              echo "<td>&nbsp;</td>";
            }
          }
          if ($formId == '7') {

            #echo  "<pre>";   print_r($row);exit;
            if (strtotime($row["PO_29"])) {
              #echo "<td>&nbsp;" . date('m/d/Y',strtotime($row["PO_29"]));

              $dt = explode(" ", $row["PO_29"]);
              echo "<td>&nbsp;" . $dt[0];
              if (in_array($row['ES_Date'], $filter_ES_date)) {
                echo " <span style='color:red'>Duplicate</span>";
              }
              else if($row['cancel_old_form_id'] != '' && $row['cancel_old_form_id'] != 0){
                echo " <span style='color:red'>Duplicate</span>";
              }
              echo "</td>\n";
            } else {
              echo "<td>&nbsp;</td>";
            }
          }
          if ($formId == '8') {
            //print_r($row);exit;
            if (strtotime($row["PT_1"])) {
              echo "<td>&nbsp;" . date('m/d/Y', strtotime($row["PT_1"]));
              if (in_array($row['ES_Date'], $filter_ES_date)) {
                echo " <span style='color:red'>Duplicate</span>";
              }
              echo "</td>\n";
            } else {
              echo "<td>&nbsp;</td>";
            }
          }
          if ($formId == '9') {
            //print_r($row);exit;
            if (strtotime($row["PO_5"])) {
              echo "<td>&nbsp;" . date('m/d/Y', strtotime($row["PO_5"]));
              if (in_array($row['ES_Date'], $filter_ES_date)) {
                echo " <span style='color:red'>Duplicate</span>";
              }
              echo "</td>\n";
            } else {
              echo "<td>&nbsp;</td>";
            }
          }

          //TD for POC freq.
          if ($formId == 2){
            if ($row["StatusId"] == 2)
              echo "<td>&nbsp;" . $row['freq'] . "</td>";
            else
              echo "<td>&nbsp;</td>";
          }
          
            //TD for MV FReq and Effective Date
          if ($formId == 6){
            $mv_freq_array = array();
            if($row["form_status"] == "submitted"){
              if($row['mv_freq'] != 'No New Orders given'){
                if($row['A'] && $row['B']){
                  $mv_freq_array[] =  $row['A'].'W'.$row['B'];
                }
                if($row['C'] && $row['D']){
                  $mv_freq_array[] =  $row['C'].'W'.$row['D'];
                }
                if($row['E'] && $row['F']){
                  $mv_freq_array[] =  $row['E'].'W'.$row['F'];
                }
                if($row['G'] && $row['H']){
                  $mv_freq_array[] =  $row['G'].'W'.$row['H'];
                }
                $mv_freq_date = date('m/d/Y',  strtotime($row['effective_date']));
              }
              else{
                $mv_freq_array[] = 'No New Orders Given';
                $mv_freq_date = 'N/A';
              }
            }
            else{
              $mv_freq_array[] = 'N/A';
              $mv_freq_date = 'N/A';
            }
            echo "<td style='width:20%;' class='text-center'>&nbsp; ".implode(',',$mv_freq_array)."</td>";
            echo "<td class='text-center'>&nbsp;{$mv_freq_date}</td>";
          }
          
          //TD for SPOF FReq and Effective Date
          if ($formId == 7){
            if ($row["StatusId"] == 2 && $row['hold'] == 1){ 
              echo "<td style='width:20%;' class='text-center'>&nbsp;Hold</td>";
              echo "<td class='text-center'>&nbsp;{$row['hold_date']}</td>";
            }
            else if($row["StatusId"] == 2 && $row['continue_freq'] == 1){
              $continue_freq_array = array();
              if($row['A'] && $row['B']){
                $continue_freq_array[] =  $row['A'].'W'.$row['B'];
              }
              if($row['C'] && $row['D']){
                $continue_freq_array[] =  $row['C'].'W'.$row['D'];
              }
              if($row['E'] && $row['F']){
                $continue_freq_array[] =  $row['E'].'W'.$row['F'];
              }
              if($row['G'] && $row['H']){
                $continue_freq_array[] =  $row['G'].'W'.$row['H'];
              }              
              echo "<td style='width:20%;' class='text-center'>&nbsp;Continue ".implode(',',$continue_freq_array)."</td>";
              echo "<td class='text-center'>&nbsp;{$row['continue_date']}</td>";
            }
            else if($row["StatusId"] == 2 &&  $row['resume_freq'] == 1){
              $resume_freq_array = array();
              if($row['A1'] && $row['B1']){
                $resume_freq_array[] =  $row['A1'].'W'.$row['B1'];
              }
              if($row['C1'] && $row['D1']){
                $resume_freq_array[] =  $row['C1'].'W'.$row['D1'];
              }
              if($row['E1'] && $row['F1']){
                $resume_freq_array[] =  $row['E1'].'W'.$row['F1'];
              }
              if($row['G1'] && $row['H1']){
                $resume_freq_array[] =  $row['G1'].'W'.$row['H1'];
              }              
              echo "<td style='width:20%;' class='text-center'>&nbsp;Resume ".implode(',',$resume_freq_array)."</td>";
              echo "<td class='text-center'>&nbsp;{$row['resume_date']}</td>";
            }
            else{
              echo "<td style='width:20%;' class='text-center'>N/A</td>";
              echo "<td class='text-center'>N/A</td>";
            }
          }
          
          //TD for form submitted or not
          

          $cancel_flag = "";
          #echo "<pre>"; print_r($row); echo "</pre>";
          if ($row["cancel_flag"] == 'yes') {
            $cancel_flag = '<span style="color:red">Cancelled</span>';
          }
          if($status == 'No'){
          echo "<td >&nbsp;" . $status . "&nbsp;" . $cancel_flag . "</td>\n";
          }else{
            echo "<td>&nbsp;" . $status . "&nbsp;" . $cancel_flag . "</td>\n";
          }
          
          if($status == 'No'){$submit_check=1;}
          
//          if ($formId == 2)
//            if ($row["StatusId"] == 2)
//              echo "<td>&nbsp;" . $row['freq'] . "</td>";
//            else
//              echo "<td>&nbsp;</td>";

          echo "</tr>\n";

          #check if dvr exists and show here
          if ($formId == 1) {
            #if its not duplicate then only check for dvr
            #echo "<pre>"; print_r($row); echo "</pre>";
            #if (!in_array($row['ES_Date'], $filter_ES_date)) {
            if ($row['cancel_flag'] == 'no') {
              $where = "dvr_date LIKE '" . date("Y-m-d", strtotime($row["TI_1"])) . "%' AND (charge_code =  'EVAL / RE-EVAL' OR charge_code =  'SOC' OR charge_code = 'ROC/RE-CERT OASIS') AND referral_id = " . $_REQUEST['referralId'];
              #echo $where."<br/>";
              $dvr_data = $curAgency->selectData('tbl_dvr', array('paper_name', 'referral_id', 'dvr_date', 'id', 'patient_sign', 'is_submitted'), $where, 'ORDER BY id ASC');
              #echo '<pre>'; print_r($dvr_data); exit;
              if (!empty($dvr_data)) {
                foreach ($dvr_data as $itemDvr) {
                  echo "<tr>";
                  echo "<td>&nbsp;</td>";


                  if ($itemDvr->paper_name != '') {
                      $sasa="https://airshometherapy.com/emradmin/upload_dvr/".$itemDvr->paper_name;
                    echo "<td>&nbsp;<a class='aLink'  href='" . $sasa . "' target='_blank'>DVR</a></td>";
                    echo "<td>&nbsp;" . date("m/d/Y", strtotime($itemDvr->dvr_date)) . "</td>";
                    echo "<td>&nbsp;" . ucfirst($itemDvr->is_submitted) . "(PaperDVR)</td>";
                  } else {
                    echo "<td>&nbsp;<a class='aLink' href='dvr.php?id=" . $itemDvr->id . "'>DVR</a></td>";
                    echo "<td>&nbsp;" . date("m/d/Y", strtotime($itemDvr->dvr_date)) . "</td>";
                    echo "<td>&nbsp;" . ucfirst($itemDvr->is_submitted) . "</td>";
                  }

                  /* if (($itemDvr->patient_sign == '0' || $itemDvr->patient_sign == '') &&  || $itemDvr->is_submitted == 'no') {
                    echo "<td>&nbsp;No</td>";
                    }else{
                    echo "<td>&nbsp;Yes</td>";

                    } */

                  echo "</tr>";
                }
              }
            }
          }



          #check if dvr exists and show here
          if ($formId == 3) {
            #if its not duplicate then only check for dvr
            #if (!in_array($row['ES_Date'], $filter_ES_date)) {
            if ($row['cancel_flag'] == 'no') {
              $where = "dvr_date LIKE '" . date("Y-m-d", strtotime($row["TI_4"])) . "%' AND (charge_code != 'EVAL / RE-EVAL' AND charge_code != 'SOC' AND charge_code != 'ROC/RE-CERT OASIS')  AND referral_id = " . $_REQUEST['referralId'];
              $dvr_data = $curAgency->selectData('tbl_dvr', array('paper_name', 'referral_id', 'id', 'dvr_date', 'patient_sign', 'is_submitted'), $where, 'ORDER BY id ASC');
              #echo "<pre>"; print_r($dvr_data); exit;
              if (!empty($dvr_data)) {
                foreach ($dvr_data as $itemDvr) {
                  #echo '<pre>'; print_r($itemDvr);
                  echo "<tr>";
                  echo "<td>&nbsp;</td>";
                  if ($itemDvr->paper_name != '') {
                      $sasa="https://airshometherapy.com/emradmin/upload_dvr/".$itemDvr->paper_name;
                    echo "<td>&nbsp;<a class='aLink'  href='" . $sasa . "' target='_blank'>DVR</a></td>";
                    echo "<td>&nbsp;" . date("m/d/Y", strtotime($itemDvr->dvr_date)) . "</td>";
                    echo "<td>&nbsp;" . ucfirst($itemDvr->is_submitted) . "(PaperDVR)</td>";
                  } else {
                    echo "<td>&nbsp;<a class='aLink' href='dvr.php?id=" . $itemDvr->id . "'>DVR</a></td>";
                    echo "<td>&nbsp;" . date("m/d/Y", strtotime($itemDvr->dvr_date)) . "</td>";
                    echo "<td>&nbsp;" . ucfirst($itemDvr->is_submitted) . "</td>";
                  }
                  /* if ($itemDvr->patient_sign == '0' || $itemDvr->patient_sign == '' || $itemDvr->is_submitted == 'no') {
                    echo "<td>&nbsp;No</td>";
                    }else{
                    echo "<td>&nbsp;Yes</td>";

                    } */

                  echo "</tr>";
                }
              }
            }
          }






          if (!in_array($row['ES_Date'], $filter_ES_date) && $row['ES_Date'] != "0000-00-00 00:00:00") {
            $filter_ES_date[] = date('Y-m-d',  strtotime($row['ES_Date']));
          }
          #echo "<pre>"; print_r($filter_ES_date);
          $arr_cnt++;
        }
        }
        if ($totalRows == 0) {
          if ($formId == '5')
            $_SESSION['flag_visit_14'] = true;
          echo "<tr>";
          echo "<td colspan='5'>";
          echo "No form(s) submitted yet.";
          echo "</td>";
          echo "</tr>";
        }
        else {
          if ($formId == '5')
            $_SESSION['flag_visit_14'] = false;
        }
        echo "</tbody>";
        echo "</table>";

        $result->close();
        ?>

<script type="text/javascript">
     
          // view fax attachment
           $('#faxattachment').click(function(e){
            e.stopImmediatePropagation();
            var pt_id = '<?=$row_refer['PatientId']?>';
            var therapist_id = '<?=$row_refer['TherapistId']?>'; 
                $.fancybox.open({
               href: "common_functions.php",
               type: "ajax",
               height: 300,
       //        width:400,
               autoSize: false,
               ajax: {
                   type: "POST",
                   data: {'func':'display_fax_attachment','patient_id':pt_id,'therapist_id':therapist_id}
               },
               });
           });
            var submit_notice = '<?php echo $submit_check;?>';
            
            var deleteFlag = '<?php echo $row_refer['deleteFlag'];?>';
            
            var dischargeFlag = '<?php echo $row_refer['dischargeFlag'];?>';
         
            if( submit_notice > 0 && dischargeFlag == 0 && deleteFlag == 0 ){
                
                 document.getElementById("notice").innerHTML = "There is unsubmitted forms. please submit it";
            }
         
      </script>

      <?php   
      }
      ?>
<style>
  .new-blank-form .bootstrap-switch { width: 82px !important;}
  .new-blank-form .bootstrap-switch-handle-on, .new-blank-form .bootstrap-switch-label, .new-blank-form .bootstrap-switch-handle-off { width: 40px !important;}
  .new-blank-form .bootstrap-switch .bootstrap-switch-handle-on, .new-blank-form .bootstrap-switch .bootstrap-switch-handle-off, .new-blank-form .bootstrap-switch .bootstrap-switch-label { padding: 4px 12px;}
  .patient-forms-accordion .sub-accordion .new-blank-form { float: right;}
</style>

<script type="text/javascript">
        //DRAG AND DROP DOCUMENT 
    var fileList = [];
    var file_up_names = [];
    var file_size = 0;
    
    Dropzone.options.myDropzone = {
        addRemoveLinks: true,
        maxFilesize: 50,
        acceptedFiles: ".pdf,.PDF",
        init: function () {
            var submitButton = document.querySelector("#submit-all");
            var cancelButton = document.querySelector("#cancel-all");
            thisDropzone = this;
            submitButton.addEventListener("click", function () {
              if(file_up_names == ''){
                  alert('Please select document first.');
                  return false;
                }
                $('#body-loader').show();
                var patient_id = $('#patient_id').val();
                var therapist_id = $('#therapist_id').val();
                url = "uploadfaxattachment.php";
                $.ajax({
                    url: url,
                    type: "POST",
                    data: ({documentArr: file_up_names, patient_id: patient_id ,therapist_id: therapist_id }),
                    success: function (data) {
                        if(data == 1){
                          $('#body-loader').hide();
                          $('#fax-msg').show();
                            setTimeout(function(){
                            $('#fax-msg').hide();
                               }, 3000);
                          file_up_names = [];
                          $("div.dz-file-preview").remove();
                          $(".dz-message").show();
                        }
                    }
                });
            });
            
            cancelButton.addEventListener("click", function(){
                $.ajax({
                    url : "delete_all_file.php",
                    type : "POST",
                    data : ({documentarr : file_up_names}),
                    success : function(data){
                        if(data == 1){
                            //alert("Files removed successfully");
                            file_up_names = [];
                            $("div.dz-file-preview").remove();
                            $(".dz-message").show();
                        }
                    }
                });
            });
            
            
            this.on("success", function (file, serverFileName) {
              if(file.name.substr(-3) == 'pdf' || file.name.substr(-3) == 'PDF'){
                fileList[serverFileName] = {"serverFileName": serverFileName, "fileName": file.name};
              }
            });
             this.on("removedfile", function (file) {
                $('.dz-progress').show();
                var count = file_up_names.length;
                if(count == 1){
                $(".dz-message").show();
                }
                file_up_names.splice(file_up_names.indexOf(file.name.toLowerCase()), 1);
                if (file_size > 0) {
                    file_size = parseFloat(file_size) - parseFloat(file.size) / 1048576;
                }
                $.ajax({
                    url: "deletedocument.php",
                    type: "POST",
                    data: {'name': file.name},
                    success: function (data) {
                        $('.dz-progress').hide();
                        
                    }
                });
            });
            this.on("complete", function (file) {
              $(".dz-message").hide();
              if(file.name.substr(-3) == 'pdf' || file.name.substr(-3) == 'PDF'){
//                file_up_names.push(file.name.toLowerCase());
                file_up_names.push(file.name);
                file_size = parseFloat(file_size) + parseFloat(file.size) / 1048576;
              }else{
                alert(file.name+' Can not uploaded');
                  return false;
              }
                
            });
            
        }
    };
    
    $(document).ready(function(){
       $("#fax-attachment").click(function () {
            $("#sucessdocument").html('');
            $("#sucessdocument").hide();
            $("#UploadBy").val('NativeDocupload');
            $('#UploadDocumentModal').modal('show');
        });
        $(".closeuploaddoc").click(function () {
            $('#UploadDocumentModal').modal('hide');
        }); 
    });
    
    $('#viewfaxattachment').click(function(e){
  e.stopImmediatePropagation();
  var therapist_id = '<?=$row_refer['TherapistId']?>';
  var pt_id = '<?=$patient_id?>'; 
      $.fancybox.open({
     href: "common_functions.php",
     type: "ajax",
     height: 300,
//        width:400,
     autoSize: false,
     ajax: {
         type: "POST",
         data: {'func':'display_fax_attachment','patient_id':pt_id,'therapist_id':therapist_id}
     },
     });
   }); 
   
   
 function deleteDocument(id) {
   var res = confirm("Are you sure you want to delete?");
   if (res) {
     var therapist_id = '<?=$row_refer['TherapistId']?>';
     var pt_id = '<?=$patient_id?>'; 
     $('#body-loader-modal').show();
     $.ajax({
          url: "deleteDocumentById.php",
          type: "POST",
          data: {'id': id},
          success: function (data) {
            if(data == 1){
          
             $.ajax({
                type: "POST",
                url: "common_functions.php",
                data: {'func':'ajax_fax_attachment','patient_id':pt_id,'therapist_id':therapist_id},
                success: function (data) {
                  $('#body-loader-modal').hide();
                  $('#fax-delete-msg').show();
                  setTimeout(function(){
                  $('#fax-delete-msg').hide();
                }, 3000);
                $('#fax-data').html('');
                  $('#fax-data').html(data);
                }
            });
            
          }
          }
      });
    }
    }
        
</script>
