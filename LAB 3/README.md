# Translations For Classify Images with Pre-built ML Models using Cloud Vision API and AutoML Lab

In this lab we upload images to Cloud Storage and use the uploaded images to train a model to to recognize different types of clouds.

Requirements
- Enable Cloud AutoML API

Create environment variables for your Project ID and Qwiklabs Username:
```
export PROJECT_ID=$DEVSHELL_PROJECT_ID
export QWIKLABS_USERNAME=student-03-59ef885546ca@qwiklabs.net
```

Create a Storage Bucket for the images used later:
```
gsutil mb -p $PROJECT_ID \
    -c regional    \
    -l us-central1 \
    gs://$PROJECT_ID-vcm/
```

Create an environment variable. Replace YOUR_BUCKET_NAME with the name of the Storage Bucket created earlier:
```
export BUCKET=YOUR_BUCKET_NAME
```

Copy training images into your bucket:
```
gsutil -m cp -r gs://automl-codelab-clouds/* gs://${BUCKET}
```

Copy the file to your Cloud Shell instance:
```
gsutil cp gs://automl-codelab-metadata/data.csv .
```

Update the CSV with the files in your project:
```
sed -i -e "s/placeholder/${BUCKET}/g" ./data.csv
```

Upload file to your Cloud Storage bucket:
```
gsutil cp ./data.csv gs://${BUCKET}
```
