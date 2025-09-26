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
</script>
