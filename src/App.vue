<template>
  <div id="app">{{ message }}</div>
  <div id="map"></div>
</template>

<script>
import maplibregl from 'maplibre-gl';
import AWS from 'aws-sdk';

export default {
  name: 'App',
  data() {
    return{
    message: "Location test"
    }
  }
}
//   
const { Signer } = window.aws_amplify_core;
const identityPoolId = String(process.env.VUE_APP_IDENTITY_POOL_ID);
const mapName = String(process.env.VUE_APP_MAP_NAME);

AWS.config.region = identityPoolId.split(":")[0];
const credentials = new AWS.CognitoIdentityCredentials({
  IdentityPoolId: identityPoolId,
});

function transformRequest(url, resourceType) {
  if (resourceType === "Style" && !url.includes("://")) {
    url = `https://maps.geo.${AWS.config.region}.amazonaws.com/maps/v0/maps/${url}/style-descriptor`;
  }
  if (url.includes("amazonaws.com")) {
    return {
      url: Signer.signUrl(url, {
        access_key: credentials.accessKeyId,
        secret_key: credentials.secretAccessKey,
        session_token: credentials.sessionToken,
      }),
    };
  }
  return { url };
}

async function initializeMap() {
  await credentials.getPromise();
  const map = new maplibregl.Map({
    container: "map",
    center: [135.75, 35.01],
    zoom: 10,
    style: mapName,
    transformRequest,
  });
  map.addControl(new maplibregl.NavigationControl(), "top-left");
}

initializeMap();
</script>

<style>
#map { height: 100vh; }
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
