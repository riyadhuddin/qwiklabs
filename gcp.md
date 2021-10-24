# intro to apis in google cloud
-> login to gcp -> Navigation menu -> api services -> Library
-> Search -> Select and enable an api 
-> client server model -> get, put, post, delete
-> GCP cloud storage json api -> cloud shell -> nano values.json -> { bucketname, location, storageclass}
-> browse https://developers.google.com/oauthplayground/ -> select and authorize api -> cloud storage api v1 -> /auth_devstorage_fullcontrol -> click authorize
-> click exchange auth code -> copy access token -> cmd ls -> export OAUTH2_TOKEN=<YOUR_TOKEN> -> export PROJECT_ID=<YOUR_PROJECT_ID>
-> create a storage bucket with this 
curl -X POST --data-binary @values.json \
    -H "Authorization: Bearer $OAUTH2_TOKEN" \
    -H "Content-Type: application/json" \
    "https://www.googleapis.com/storage/v1/b?project=$PROJECT_ID"
-> bucket was created with json api -> realpath nosepin.JPG ->output: /home/student_04_48b495a05486/nosepin.JPG
-> export OBJECT=/home/student_04_48b495a05486/nosepin.JPG -> export BUCKET_NAME=ctg-dhaka1
-> curl -X POST --data-binary @$OBJECT \
    -H "Authorization: Bearer $OAUTH2_TOKEN" \
    -H "Content-Type: image/png" \
    "https://www.googleapis.com/upload/storage/v1/b/$BUCKET_NAME/o?uploadType=media&name=demo-image"
# Extract, Analyze, and Translate Text from Images with the Cloud ML APIs 
-> Activate cloud shell -> gcloud auth list -> gcloud config list project -> apis -> create credentials -> api key & copy
-> export API_KEY=<YOUR_API_KEY>
-> cloud storage -> create bucket -> control access object -> upload image -> edit permission -> public all reader 
-> nano ocr-request.json
-> Text-detection nano ocr-request.json
{
  "requests": [
      {
        "image": {
          "source": {
              "gcsImageUri": "gs://my-bucket-name/sign.jpg"
          }
        },
        "features": [
          {
            "type": "TEXT_DETECTION",
            "maxResults": 10
          }
        ]
      }
  ]
}
-> curl -s -X POST -H "Content-Type: application/json" --data-binary @ocr-request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}
-.> save response curl -s -X POST -H "Content-Type: application/json" --data-binary @ocr-request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY} -o ocr-response.json
->  nano translation-request.json
{
  "q": "your_text_here",
  "target": "en"
}
-> STR=$(jq .responses[0].textAnnotations[0].description ocr-response.json) && STR="${STR//\"}" && sed -i "s|your_text_here|$STR|g" translation-request.json
-> curl -s -X POST -H "Content-Type: application/json" --data-binary @translation-request.json https://translation.googleapis.com/language/translate/v2?key=${API_KEY} -o translation-response.json
-> cat translation-response.json
-> nano nl-request.json 
{
  "document":{
    "type":"PLAIN_TEXT",
    "content":"your_text_here"
  },
  "encodingType":"UTF8"
}
-> STR=$(jq .data.translations[0].translatedText  translation-response.json) && STR="${STR//\"}" && sed -i "s|your_text_here|$STR|g" nl-request.json
-> curl "https://language.googleapis.com/v1/documents:analyzeEntities?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @nl-request.json
# Classify Text into Categories with the Natural Language API 
