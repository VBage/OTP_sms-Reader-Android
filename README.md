# OTP_sms-Reader-Android
reads User's incoming OTP Reading for Android, Reading incoming SMS messages for verification 

You need to use Broadcast Receiver to perform it.

you can download Source Code Here

 

Add this code to your Manifest.XML

<uses-permission android:name="android.permission.RECEIVE_SMS" />
<uses-permission android:name="android.permission.READ_SMS" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
 

<receiver android:name=".receivers.OTPBroadCastReceiver">
    <intent-filter android:priority="999">
        <action android:name="android.provider.Telephony.SMS_RECEIVED" />
    </intent-filter>
</receiver>
Create MainActivity.java


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        otpOnlyTextView = (TextView) findViewById(R.id.opt_textView_ID);
        fullmessageTextView = (TextView) findViewById(R.id.fullmessage_textView_ID);
        Button sendMsg=(Button)findViewById(R.id.sendMsg_ID);
        sendMsg.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {

                //sendMessageAPILINK(); USE ANY API TO SEND MESSAGE OR SEND ANY NORMAL MESSAGE THROUGH YOUR ANOTHER NUMBER
                checkAndRequestPermissions();
            }
        });
    }

    private BroadcastReceiver receiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            if (intent.getAction().equalsIgnoreCase("otp")) {
                final String message = intent.getStringExtra("message");
                final String sender = intent.getStringExtra("Sender");
                otpOnlyTextView.setText(message.replaceAll("\\D+", ""));
                fullmessageTextView.setText(sender + " : " + message);
                Toast.makeText(getBaseContext(), message, Toast.LENGTH_LONG).show();
                Log.e("OTP MESSSGE", message);
            }
        }
    };

    private boolean checkAndRequestPermissions() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M && checkSelfPermission(android.Manifest.permission.READ_SMS) != PackageManager.PERMISSION_GRANTED) {
            int receiveSMS = ContextCompat.checkSelfPermission(this, android.Manifest.permission.RECEIVE_SMS);
            int readSMS = ContextCompat.checkSelfPermission(this, android.Manifest.permission.READ_SMS);
            List<String> listPermissionsNeeded = new ArrayList<>();
            if (receiveSMS != PackageManager.PERMISSION_GRANTED) {
                listPermissionsNeeded.add(Manifest.permission.RECEIVE_SMS);
            }
            if (readSMS != PackageManager.PERMISSION_GRANTED) {
                listPermissionsNeeded.add(android.Manifest.permission.READ_SMS);
            }
            if (!listPermissionsNeeded.isEmpty()) {
                ActivityCompat.requestPermissions(this,
                        listPermissionsNeeded.toArray(new String[listPermissionsNeeded.size()]), 1);
                return false;
            }
            return true;
        }
        return true;

    }

    @Override
    public void onResume() {
        LocalBroadcastManager.getInstance(this).registerReceiver(receiver, new IntentFilter("otp"));
        super.onResume();
    }

    @Override
    public void onPause() {
        super.onPause();
        LocalBroadcastManager.getInstance(this).unregisterReceiver(receiver);
    }


 

create  receivers.OTPBroadCastReceiver.java

    @Override
    public void onReceive(Context context, Intent intent)
    {
        // Get Bundle object contained in the SMS intent passed in
        Bundle bundle = intent.getExtras();
        SmsMessage[] smsMsg = null;
        String smsStr ="";
        if (bundle != null)
        {
            // Get the SMS message
            Object[] pdus = (Object[]) bundle.get("pdus");
            smsMsg = new SmsMessage[pdus.length];
            for (int i=0; i<smsMsg.length; i++){
                smsMsg[i] = SmsMessage.createFromPdu((byte[])pdus[i]);
                smsStr = smsMsg[i].getMessageBody().toString();

                String Sender = smsMsg[i].getOriginatingAddress();
                //Check here sender is yours
                Intent smsIntent = new Intent("otp");
                smsIntent.putExtra("message",smsStr);
                smsIntent.putExtra("Sender",Sender);
                LocalBroadcastManager.getInstance(context).sendBroadcast(smsIntent);
            }
        }
    }
