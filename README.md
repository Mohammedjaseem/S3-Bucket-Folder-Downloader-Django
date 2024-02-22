        import boto3
        import zipfile
        from io import BytesIO  # Import BytesIO from the io module
        from django.http import HttpResponse
        from django.views import View
        
        class DownloadFolderView(View):
            def get(self, request):
                # Replace 'your-bucket-name' and 'your-folder-path' with your actual bucket name and folder path
                bucket_name = 'connectv2s3'
                folder_path = 'Inceptra2k24/QrCodes/'
        
                # Initialize Boto3 S3 client
                s3 = boto3.client('s3')
        
                try:
                    # List objects in the folder
                    objects = s3.list_objects_v2(Bucket=bucket_name, Prefix=folder_path)
        
                    # Generate a zip file in memory containing the folder contents
                    with BytesIO() as zip_buffer:
                        with zipfile.ZipFile(zip_buffer, 'a', zipfile.ZIP_DEFLATED, False) as zip_file:
                            for obj in objects['Contents']:
                                # Skip if the object is a folder itself
                                if not obj['Key'].endswith('/'):
                                    # Download each object in the folder
                                    response = s3.get_object(Bucket=bucket_name, Key=obj['Key'])
                                    # Write the object to the zip file
                                    zip_file.writestr(obj['Key'].replace(folder_path, '', 1), response['Body'].read())
        
                        # Prepare the response with the zip file
                        response = HttpResponse(zip_buffer.getvalue(), content_type='application/zip')
                        response['Content-Disposition'] = f'attachment; filename="{folder_path}.zip"'
                        return response
        
                except Exception as e:
                    # Handle any errors (e.g., bucket not found, folder not found, etc.)
                    return HttpResponse(f"An error occurred: {e}", status=500)
