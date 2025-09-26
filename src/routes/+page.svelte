<script>
    import { onMount, onDestroy } from "svelte"; //onDestroy is useed to release the camera and close the websocket when the component is closed
    let camera = null; //camera will hold the media stream
    let socket = null; //socket will hold the websocket connection to your server
    let recorder = null; //recorder will hold the MediaRecorder used to produce small video/audio blobs
    let videoelement;
    let ipAddress = "";
    let status = "Ready to connect";
    let isConnecting = false;
    //ipAddress, status and isConnecting are UI state variables

    async function connect() {
        const serverAddress = ipAddress.trim();
        if (!serverAddress) {
            alert("Please enter server IP address and port");
            return;
        }
    }
    isConnecting= true;
    status='Initializing Camera...';
    try{
        camera = await navigator.mediaDevices.getUserMedia({video:true, audio:true});
        //asks browser for camera and microphone access
    console.log("Inspecting tracks in the media stream");
    camera.getTracks().forEach(track => console.log(`Found track: kind=${track.kind},label=$track.label`))//logs each track
    };

</script>
