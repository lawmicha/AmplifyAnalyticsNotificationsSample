# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'
# Update this to point to your local amplify-ios repository
$LOCAL_REPO = '~/aws-amplify/amplify-ios'

target 'AmplifyAnalyticsNotificationSample' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  # Pods for AmplifyAnalyticsNotificationSample
  # Amplify (explicit dependencies required when using pods with local path)
  pod 'Amplify', :path => $LOCAL_REPO
  pod 'AmplifyPlugins', :path => $LOCAL_REPO # No need for this when not using local pods
  pod 'AWSPluginsCore', :path => $LOCAL_REPO # No need for this when not using local pods

  pod 'AmplifyPlugins/AWSPinpointAnalyticsPlugin', :path => $LOCAL_REPO
  pod 'AmplifyPlugins/AWSCognitoAuthPlugin', :path => $LOCAL_REPO
  
  target 'AmplifyAnalyticsNotificationSampleTests' do
    inherit! :search_paths
    # Pods for testing
  end

end
