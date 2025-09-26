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
    isConnecting = true;
    status = "Initializing Camera...";
    try {
        camera = await navigator.mediaDevices.getUserMedia({
            video: true,
            audio: true,
        });
        //asks browser for camera and microphone access
        console.log("Inspecting tracks in the media stream");
        camera
            .getTracks()
            .forEach((track) =>
                console.log(
                    `Found track: kind=${track.kind},label=$track.label`,
                ),
            ); //logs each track
        videoelement.srcObject = camera; //sets the video element to show the camera stream live
    } catch (err) {
        //if user denies permission or there is no camera catch runs , logs error and updates status,resets isConnecting and exits connect()
        console.error("getUserMedia error:", err);
        status = "Error: Could not access camera.Please grant permission";
        isConnecting = false;
        return;
    }
    videoElement.onloadedmetadata = () => {
        const clientId = "client_" + Math.floor(Math.random() * 10000); //Random Client ID generated
        const wsUrl = `ws://${serverAddress}/${clientId}`; //Builds a web socket URL
        socket = new WebSocket(wsUrl); //Creates a new WebSocket connection to server
        socket.onopen = () => {
            //when connection opens update status
            status = "Connected";
            isConnecting = false;
        };
    };
    socket.onmessage = (event) => {
        const message =JSON.parse(event data);//parses incoming message
        if (message.command === 'start_recording') { //if command is start_recording call startRecording function
            startRecording();
        }
        if(message.command === 'stop_recording'){//if command is stop_recording call stopRecording function
            stopRecording();
        }
    };
    socket.onclose = () => {//runs when socket closes
        status = 'Connection Failed.';
        isConnecting = false;
    };
       socket.onerror = (err) => {//runs on websocket error
        console.error('WebSocket error:', err);
        status = 'Connection Failed.';
        isConnecting = false;
    };
    function startRecording() {
       if(recorder && recorder.state === 'recording'){//if already recording do nothing
        return;
       }
       status='Recording...';
       if(!camera) return;//if camera is missing for some reason,exit
       recorder = new MediaRecorder(camera,{mimeType:'video/webm'});//create a new MediaRecorder that records from camera mediastream
       recorder.ondataavailable = (event) => {//fires when a chunk of recorded data is ready
        if(event.data && event.data.size > 0 ){
            socket.send(event.data);//sends the recorded blob to server via websocket
        }
    };
        recorder.onstart=() => {//logs when recording starts
        console.log('MediaRecorder has started');
        }
        recorder.onstop = () => {//logs when recording stops
            console.log('MediaRecorder has stopped.Notifying server');
            status='Connected';//update the UI status
            if(socket && socket.readyState === WebSocket.OPEN){
                socket.send(JSON.stringify({type:'recording_fully_stopped'}));//if socket is open,notify server that recording has stopped
            }
        }
        recorder.start(1000);//starts recording and asks the recorder to deliver ondataavailable events every 1000 millisecond
    }
    function stopRecording(){
        if(recorder && recorder.state === 'recording'){//if recorder exists and is recording
            recorder.stop();//stop the recording
        }
    }
    onDestroy(() => {
       
    });

</script>
