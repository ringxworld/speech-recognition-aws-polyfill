speech-recognition-aws-polyfill
===
Forked off [speech-recognition-aws-polyfill](https://github.com/ceuk/speech-recognition-aws-polyfill) to get working with  [react-speech-recognition](https://github.com/JamesBrill/react-speech-recognition)


## Who is it for?

This Library is a good fit if you are already using AWS services (or you would just prefer to use AWS).

A polyfill also exists at: [/antelow/speech-polyfill](https://github.com/anteloe/speech-polyfill), which uses [Azure Cognitive Services](https://azure.microsoft.com/en-gb/services/cognitive-services/) as a fallback. However, it seems to have gone stale with no updates for ~2 years.


## Prerequisites

* An AWS account
* A Cognito [identity pool](https://docs.aws.amazon.com/cognito/latest/developerguide/identity-pools.html) (unauthenticated or authenticated) with the `TranscribeStreaming` permission.

## AWS Setup Guide

1. In the AWS console, visit the Cognito section and click **Manage Identity Pools**.
1. Click **Create new identity pool** and give it a name.
1. To allow anyone who visits your app to use speech recognition (e.g. for public-facing web apps) check **Enable access to unauthenticated identities**
1. If you want to configure authentication instead, do so now.
1. Click **Create Pool**
1. Choose or create a role for your users. If you are just using authenticated sessions, you are only interested in the second section. If you aren't sure what to do here, the default role is fine.
1. Make sure your role has the `TranscribeStreaming` policy attached. To attach this to your role search for IAM -> Roles, find your role, click "Attach policies" and search for the TranscribeStreaming role.
1. Go back to Cognito and find your identity pool. Click **Edit identity pool** in the top right and make a note of your **Identity pool ID**

## Usage

Install with `npm i --save speech-recognition-aws-polyfill`

Usage with [react-speech-recognition](https://github.com/JamesBrill/react-speech-recognition)
```javascript
import React, { useState } from 'react'
import SpeechTranscribeCallbackComponent from './transcribe/SpeechTranscribeCallback'
import SpeechRecognition, { useSpeechRecognition } from 'react-speech-recognition';
import SpeechRecognitionPolyfill from 'speech-recognition-aws-polyfill'

let isChrome = true;
try{
  isChrome = !!window.chrome || (!!window.chrome.webstore || !!window.chrome.runtime);
}catch(err){
  isChrome = false;
}


const SpeechRecognitionPoly = (!isChrome) ? SpeechRecognititionPolyfill.create({
      IdentityPoolId: '<--- Pool id --->', 
      region: '<--- region --->',
      lang: 'en-US'
    }) : null;

if(SpeechRecognitionPoly !== null){
  SpeechRecognition.applyPolyfill(SpeechRecognitionPoly);
}


class SpeechTranscriber extends React.Component{
    constructor(props){
      super(props);

      this.state = {
        broadcasting: false,
      }

    }

  listenOnce = () => {
      this.setState({broadcasting: true}, function(){
        SpeechRecognition.startListening({ continuous: false })
      });
  };

  stopListening = () => {
    console.log("Stoped Listening");
    this.setState({broadcasting: false}, function(){
      SpeechRecognition.stopListening();
    });
  }

render(){
    let broadcasting = (this.state.broadcasting) ? "playing" : "inactive";
    return(
        <div>
          <SpeechTranscribeCallbackComponent broadcasting={this.state.broadcasting}/>
          <button 
            id="microphone" 
            className={broadcasting}
            onMouseDown={this.listenOnce} 
            onMouseUp={this.stopListening} />   
        </div>

      );
  }
}

export default SpeechTranscriber;

```

