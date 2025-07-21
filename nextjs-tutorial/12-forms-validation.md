# บทที่ 12: Forms and Validation

## 🎯 จุดประสงค์ของบทเรียน
- เรียนรู้ Form handling ใน Next.js
- ทำความเข้าใจ Client และ Server-side validation
- เรียนรู้ React Hook Form integration
- เข้าใจ File upload และ Form optimization

## 📝 Form Fundamentals

### **🏗️ Basic Form Structure**

```jsx
// components/forms/BasicForm.js
import { useState } from 'react';

export function BasicForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });
  const [errors, setErrors] = useState({});
  const [loading, setLoading] = useState(false);

  // ✅ Handle input change
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));

    // ✅ Clear error on input change
    if (errors[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: ''
      }));
    }
  };

  // ✅ Client-side validation
  const validateForm = () => {
    const newErrors = {};

    // Name validation
    if (!formData.name.trim()) {
      newErrors.name = 'Name is required';
    } else if (formData.name.trim().length < 2) {
      newErrors.name = 'Name must be at least 2 characters';
    }

    // Email validation
    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email)) {
      newErrors.email = 'Please enter a valid email address';
    }

    // Message validation
    if (!formData.message.trim()) {
      newErrors.message = 'Message is required';
    } else if (formData.message.trim().length < 10) {
      newErrors.message = 'Message must be at least 10 characters';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  // ✅ Handle form submission
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!validateForm()) {
      return;
    }

    setLoading(true);

    try {
      const response = await fetch('/api/contact', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(formData)
      });

      const result = await response.json();

      if (response.ok) {
        // ✅ Success handling
        alert('Form submitted successfully!');
        setFormData({ name: '', email: '', message: '' });
      } else {
        // ✅ Server error handling
        if (result.errors) {
          setErrors(result.errors);
        } else {
          alert(result.message || 'Something went wrong');
        }
      }
    } catch (error) {
      console.error('Submit error:', error);
      alert('Network error occurred');
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="max-w-md mx-auto space-y-6">
      {/* Name field */}
      <div>
        <label htmlFor="name" className="block text-sm font-medium text-gray-700">
          Name *
        </label>
        <input
          type="text"
          id="name"
          name="name"
          value={formData.name}
          onChange={handleChange}
          className={`mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
            errors.name 
              ? 'border-red-300 focus:border-red-500' 
              : 'border-gray-300 focus:border-blue-500'
          }`}
          placeholder="Enter your name"
        />
        {errors.name && (
          <p className="mt-1 text-sm text-red-600">{errors.name}</p>
        )}
      </div>

      {/* Email field */}
      <div>
        <label htmlFor="email" className="block text-sm font-medium text-gray-700">
          Email *
        </label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          className={`mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
            errors.email 
              ? 'border-red-300 focus:border-red-500' 
              : 'border-gray-300 focus:border-blue-500'
          }`}
          placeholder="Enter your email"
        />
        {errors.email && (
          <p className="mt-1 text-sm text-red-600">{errors.email}</p>
        )}
      </div>

      {/* Message field */}
      <div>
        <label htmlFor="message" className="block text-sm font-medium text-gray-700">
          Message *
        </label>
        <textarea
          id="message"
          name="message"
          rows={4}
          value={formData.message}
          onChange={handleChange}
          className={`mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
            errors.message 
              ? 'border-red-300 focus:border-red-500' 
              : 'border-gray-300 focus:border-blue-500'
          }`}
          placeholder="Enter your message"
        />
        {errors.message && (
          <p className="mt-1 text-sm text-red-600">{errors.message}</p>
        )}
      </div>

      {/* Submit button */}
      <button
        type="submit"
        disabled={loading}
        className="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed"
      >
        {loading ? (
          <div className="flex items-center">
            <svg className="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
              <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            Sending...
          </div>
        ) : (
          'Send Message'
        )}
      </button>
    </form>
  );
}
```

## 🪝 React Hook Form

### **⚡ Advanced Form with React Hook Form**

```jsx
// components/forms/HookForm.js
import { useForm, Controller } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';
import { useState } from 'react';

// ✅ Yup validation schema
const schema = yup.object({
  firstName: yup
    .string()
    .required('First name is required')
    .min(2, 'First name must be at least 2 characters'),
  
  lastName: yup
    .string()
    .required('Last name is required')
    .min(2, 'Last name must be at least 2 characters'),
  
  email: yup
    .string()
    .required('Email is required')
    .email('Please enter a valid email address'),
  
  phone: yup
    .string()
    .required('Phone number is required')
    .matches(/^[0-9+\-\s()]+$/, 'Please enter a valid phone number'),
  
  dateOfBirth: yup
    .date()
    .required('Date of birth is required')
    .max(new Date(), 'Date of birth cannot be in the future'),
  
  country: yup
    .string()
    .required('Please select a country'),
  
  interests: yup
    .array()
    .min(1, 'Please select at least one interest'),
  
  newsletter: yup
    .boolean(),
  
  terms: yup
    .boolean()
    .oneOf([true], 'You must accept the terms and conditions'),
  
  bio: yup
    .string()
    .max(500, 'Bio must not exceed 500 characters')
});

const countries = [
  { value: 'US', label: 'United States' },
  { value: 'CA', label: 'Canada' },
  { value: 'UK', label: 'United Kingdom' },
  { value: 'TH', label: 'Thailand' },
  { value: 'JP', label: 'Japan' }
];

const interestOptions = [
  { value: 'technology', label: 'Technology' },
  { value: 'design', label: 'Design' },
  { value: 'business', label: 'Business' },
  { value: 'health', label: 'Health & Fitness' },
  { value: 'travel', label: 'Travel' },
  { value: 'food', label: 'Food & Cooking' }
];

export function HookForm() {
  const [submitStatus, setSubmitStatus] = useState(null);

  const {
    register,
    handleSubmit,
    control,
    watch,
    formState: { errors, isSubmitting, dirtyFields, isValid },
    reset,
    setValue,
    getValues
  } = useForm({
    resolver: yupResolver(schema),
    defaultValues: {
      firstName: '',
      lastName: '',
      email: '',
      phone: '',
      dateOfBirth: '',
      country: '',
      interests: [],
      newsletter: false,
      terms: false,
      bio: ''
    },
    mode: 'onChange' // Validate on change
  });

  // ✅ Watch specific fields
  const watchedInterests = watch('interests');
  const watchedCountry = watch('country');

  // ✅ Form submission
  const onSubmit = async (data) => {
    try {
      setSubmitStatus('loading');

      const response = await fetch('/api/user/profile', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(data)
      });

      if (response.ok) {
        setSubmitStatus('success');
        reset(); // Reset form after successful submission
      } else {
        const error = await response.json();
        setSubmitStatus('error');
        console.error('Submission error:', error);
      }
    } catch (error) {
      setSubmitStatus('error');
      console.error('Network error:', error);
    }
  };

  // ✅ Custom input component
  const Input = ({ label, name, type = 'text', required = false, ...props }) => (
    <div>
      <label htmlFor={name} className="block text-sm font-medium text-gray-700">
        {label} {required && <span className="text-red-500">*</span>}
      </label>
      <input
        id={name}
        type={type}
        {...register(name)}
        className={`mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
          errors[name] 
            ? 'border-red-300 focus:border-red-500' 
            : 'border-gray-300 focus:border-blue-500'
        }`}
        {...props}
      />
      {errors[name] && (
        <p className="mt-1 text-sm text-red-600">{errors[name].message}</p>
      )}
    </div>
  );

  // ✅ Custom select component
  const Select = ({ label, name, options, required = false, ...props }) => (
    <div>
      <label htmlFor={name} className="block text-sm font-medium text-gray-700">
        {label} {required && <span className="text-red-500">*</span>}
      </label>
      <select
        id={name}
        {...register(name)}
        className={`mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
          errors[name] 
            ? 'border-red-300 focus:border-red-500' 
            : 'border-gray-300 focus:border-blue-500'
        }`}
        {...props}
      >
        <option value="">Select {label.toLowerCase()}</option>
        {options.map((option) => (
          <option key={option.value} value={option.value}>
            {option.label}
          </option>
        ))}
      </select>
      {errors[name] && (
        <p className="mt-1 text-sm text-red-600">{errors[name].message}</p>
      )}
    </div>
  );

  // ✅ Checkbox group component
  const CheckboxGroup = ({ label, name, options, required = false }) => (
    <div>
      <fieldset>
        <legend className="block text-sm font-medium text-gray-700">
          {label} {required && <span className="text-red-500">*</span>}
        </legend>
        <div className="mt-2 space-y-2">
          {options.map((option) => (
            <div key={option.value} className="flex items-center">
              <input
                type="checkbox"
                id={`${name}-${option.value}`}
                value={option.value}
                {...register(name)}
                className="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded"
              />
              <label 
                htmlFor={`${name}-${option.value}`}
                className="ml-2 text-sm text-gray-700"
              >
                {option.label}
              </label>
            </div>
          ))}
        </div>
      </fieldset>
      {errors[name] && (
        <p className="mt-1 text-sm text-red-600">{errors[name].message}</p>
      )}
    </div>
  );

  return (
    <div className="max-w-2xl mx-auto p-6">
      <h2 className="text-2xl font-bold text-gray-900 mb-6">User Profile Form</h2>
      
      {/* Success/Error Messages */}
      {submitStatus === 'success' && (
        <div className="mb-6 p-4 bg-green-50 border border-green-200 rounded-md">
          <p className="text-green-800">Profile updated successfully!</p>
        </div>
      )}
      
      {submitStatus === 'error' && (
        <div className="mb-6 p-4 bg-red-50 border border-red-200 rounded-md">
          <p className="text-red-800">An error occurred. Please try again.</p>
        </div>
      )}

      <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
        {/* Personal Information */}
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          <Input
            label="First Name"
            name="firstName"
            required
            placeholder="Enter your first name"
          />
          
          <Input
            label="Last Name"
            name="lastName"
            required
            placeholder="Enter your last name"
          />
        </div>

        <Input
          label="Email Address"
          name="email"
          type="email"
          required
          placeholder="Enter your email address"
        />

        <Input
          label="Phone Number"
          name="phone"
          type="tel"
          required
          placeholder="Enter your phone number"
        />

        <Input
          label="Date of Birth"
          name="dateOfBirth"
          type="date"
          required
        />

        <Select
          label="Country"
          name="country"
          options={countries}
          required
        />

        {/* Interests */}
        <CheckboxGroup
          label="Interests"
          name="interests"
          options={interestOptions}
          required
        />

        {/* Bio */}
        <div>
          <label htmlFor="bio" className="block text-sm font-medium text-gray-700">
            Bio
          </label>
          <Controller
            name="bio"
            control={control}
            render={({ field }) => (
              <textarea
                {...field}
                id="bio"
                rows={4}
                className={`mt-1 block w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
                  errors.bio 
                    ? 'border-red-300 focus:border-red-500' 
                    : 'border-gray-300 focus:border-blue-500'
                }`}
                placeholder="Tell us about yourself"
              />
            )}
          />
          {errors.bio && (
            <p className="mt-1 text-sm text-red-600">{errors.bio.message}</p>
          )}
        </div>

        {/* Checkboxes */}
        <div className="space-y-3">
          <div className="flex items-center">
            <input
              type="checkbox"
              id="newsletter"
              {...register('newsletter')}
              className="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded"
            />
            <label htmlFor="newsletter" className="ml-2 text-sm text-gray-700">
              Subscribe to newsletter
            </label>
          </div>

          <div className="flex items-center">
            <input
              type="checkbox"
              id="terms"
              {...register('terms')}
              className="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded"
            />
            <label htmlFor="terms" className="ml-2 text-sm text-gray-700">
              I agree to the <a href="/terms" className="text-blue-600 hover:text-blue-500">terms and conditions</a> *
            </label>
          </div>
          {errors.terms && (
            <p className="mt-1 text-sm text-red-600">{errors.terms.message}</p>
          )}
        </div>

        {/* Form Actions */}
        <div className="flex space-x-4">
          <button
            type="submit"
            disabled={isSubmitting || !isValid}
            className="flex-1 bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            {isSubmitting ? 'Saving...' : 'Save Profile'}
          </button>
          
          <button
            type="button"
            onClick={() => reset()}
            className="flex-1 bg-gray-300 text-gray-700 py-2 px-4 rounded-md hover:bg-gray-400 focus:outline-none focus:ring-2 focus:ring-gray-500"
          >
            Reset
          </button>
        </div>

        {/* Debug info (development only) */}
        {process.env.NODE_ENV === 'development' && (
          <div className="mt-8 p-4 bg-gray-100 rounded-md">
            <h3 className="text-sm font-medium text-gray-700 mb-2">Debug Info:</h3>
            <p className="text-xs text-gray-600">
              Dirty fields: {Object.keys(dirtyFields).join(', ') || 'None'}
            </p>
            <p className="text-xs text-gray-600">
              Form valid: {isValid ? 'Yes' : 'No'}
            </p>
            <p className="text-xs text-gray-600">
              Selected interests: {watchedInterests?.join(', ') || 'None'}
            </p>
          </div>
        )}
      </form>
    </div>
  );
}
```

## 📤 File Upload

### **📁 File Upload Component**

```jsx
// components/forms/FileUpload.js
import { useState, useRef, useCallback } from 'react';
import { useDropzone } from 'react-dropzone';

export function FileUpload({ 
  onUpload, 
  accept = 'image/*',
  maxSize = 5 * 1024 * 1024, // 5MB
  multiple = false,
  className = ''
}) {
  const [files, setFiles] = useState([]);
  const [uploading, setUploading] = useState(false);
  const [uploadProgress, setUploadProgress] = useState({});

  // ✅ Handle file drop
  const onDrop = useCallback((acceptedFiles, rejectedFiles) => {
    // Handle rejected files
    if (rejectedFiles.length > 0) {
      rejectedFiles.forEach(({ file, errors }) => {
        errors.forEach(error => {
          console.error(`File ${file.name}: ${error.message}`);
          alert(`File ${file.name}: ${error.message}`);
        });
      });
    }

    // Process accepted files
    const newFiles = acceptedFiles.map(file => ({
      file,
      id: Math.random().toString(36).substr(2, 9),
      preview: file.type.startsWith('image/') ? URL.createObjectURL(file) : null,
      progress: 0,
      status: 'pending' // pending, uploading, success, error
    }));

    setFiles(prev => multiple ? [...prev, ...newFiles] : newFiles);
  }, [multiple]);

  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop,
    accept: {
      [accept]: []
    },
    maxSize,
    multiple
  });

  // ✅ Upload single file
  const uploadFile = async (fileObj) => {
    const formData = new FormData();
    formData.append('file', fileObj.file);
    formData.append('fileId', fileObj.id);

    try {
      // Update status
      setFiles(prev => prev.map(f => 
        f.id === fileObj.id 
          ? { ...f, status: 'uploading' }
          : f
      ));

      const xhr = new XMLHttpRequest();

      // ✅ Track upload progress
      xhr.upload.addEventListener('progress', (e) => {
        if (e.lengthComputable) {
          const progress = Math.round((e.loaded / e.total) * 100);
          setUploadProgress(prev => ({
            ...prev,
            [fileObj.id]: progress
          }));
        }
      });

      // ✅ Handle upload completion
      xhr.addEventListener('load', () => {
        if (xhr.status === 200) {
          const response = JSON.parse(xhr.responseText);
          
          setFiles(prev => prev.map(f => 
            f.id === fileObj.id 
              ? { ...f, status: 'success', url: response.url }
              : f
          ));

          if (onUpload) {
            onUpload(response);
          }
        } else {
          setFiles(prev => prev.map(f => 
            f.id === fileObj.id 
              ? { ...f, status: 'error' }
              : f
          ));
        }
      });

      // ✅ Handle upload error
      xhr.addEventListener('error', () => {
        setFiles(prev => prev.map(f => 
          f.id === fileObj.id 
            ? { ...f, status: 'error' }
            : f
        ));
      });

      xhr.open('POST', '/api/upload');
      xhr.send(formData);

    } catch (error) {
      console.error('Upload error:', error);
      setFiles(prev => prev.map(f => 
        f.id === fileObj.id 
          ? { ...f, status: 'error' }
          : f
      ));
    }
  };

  // ✅ Upload all files
  const uploadAll = async () => {
    setUploading(true);
    
    const pendingFiles = files.filter(f => f.status === 'pending');
    
    for (const file of pendingFiles) {
      await uploadFile(file);
    }
    
    setUploading(false);
  };

  // ✅ Remove file
  const removeFile = (fileId) => {
    setFiles(prev => {
      const updated = prev.filter(f => f.id !== fileId);
      
      // Revoke preview URL to prevent memory leaks
      const fileToRemove = prev.find(f => f.id === fileId);
      if (fileToRemove?.preview) {
        URL.revokeObjectURL(fileToRemove.preview);
      }
      
      return updated;
    });

    // Remove from progress tracking
    setUploadProgress(prev => {
      const updated = { ...prev };
      delete updated[fileId];
      return updated;
    });
  };

  // ✅ Format file size
  const formatFileSize = (bytes) => {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  };

  return (
    <div className={`w-full ${className}`}>
      {/* Drop Zone */}
      <div
        {...getRootProps()}
        className={`border-2 border-dashed rounded-lg p-6 text-center cursor-pointer transition-colors ${
          isDragActive
            ? 'border-blue-500 bg-blue-50'
            : 'border-gray-300 hover:border-gray-400'
        }`}
      >
        <input {...getInputProps()} />
        
        <div className="space-y-2">
          <svg
            className="mx-auto h-12 w-12 text-gray-400"
            stroke="currentColor"
            fill="none"
            viewBox="0 0 48 48"
          >
            <path
              d="M28 8H12a4 4 0 00-4 4v20m32-12v8m0 0v8a4 4 0 01-4 4H12a4 4 0 01-4-4v-4m32-4l-3.172-3.172a4 4 0 00-5.656 0L28 28M8 32l9.172-9.172a4 4 0 015.656 0L28 28m0 0l4 4m4-24h8m-4-4v8m-12 4h.02"
              strokeWidth={2}
              strokeLinecap="round"
              strokeLinejoin="round"
            />
          </svg>
          
          <div>
            <p className="text-sm text-gray-600">
              {isDragActive
                ? 'Drop the files here...'
                : 'Drag & drop files here, or click to select'
              }
            </p>
            <p className="text-xs text-gray-500">
              {accept} files up to {formatFileSize(maxSize)}
            </p>
          </div>
        </div>
      </div>

      {/* File List */}
      {files.length > 0 && (
        <div className="mt-4 space-y-2">
          <div className="flex justify-between items-center">
            <h3 className="text-sm font-medium text-gray-700">
              Files ({files.length})
            </h3>
            
            {files.some(f => f.status === 'pending') && (
              <button
                onClick={uploadAll}
                disabled={uploading}
                className="text-sm bg-blue-600 text-white px-3 py-1 rounded hover:bg-blue-700 disabled:opacity-50"
              >
                {uploading ? 'Uploading...' : 'Upload All'}
              </button>
            )}
          </div>

          <div className="space-y-2">
            {files.map((fileObj) => (
              <div
                key={fileObj.id}
                className="flex items-center space-x-3 p-3 bg-gray-50 rounded-lg"
              >
                {/* File Preview/Icon */}
                <div className="flex-shrink-0">
                  {fileObj.preview ? (
                    <img
                      src={fileObj.preview}
                      alt={fileObj.file.name}
                      className="h-10 w-10 object-cover rounded"
                    />
                  ) : (
                    <div className="h-10 w-10 bg-gray-200 rounded flex items-center justify-center">
                      <svg className="h-6 w-6 text-gray-400" fill="currentColor" viewBox="0 0 20 20">
                        <path fillRule="evenodd" d="M4 4a2 2 0 012-2h4.586A2 2 0 0112 2.586L15.414 6A2 2 0 0116 7.414V16a2 2 0 01-2 2H6a2 2 0 01-2-2V4z" clipRule="evenodd" />
                      </svg>
                    </div>
                  )}
                </div>

                {/* File Info */}
                <div className="flex-1 min-w-0">
                  <p className="text-sm font-medium text-gray-900 truncate">
                    {fileObj.file.name}
                  </p>
                  <p className="text-xs text-gray-500">
                    {formatFileSize(fileObj.file.size)}
                  </p>
                  
                  {/* Progress Bar */}
                  {fileObj.status === 'uploading' && (
                    <div className="mt-1">
                      <div className="bg-gray-200 rounded-full h-2">
                        <div
                          className="bg-blue-600 h-2 rounded-full transition-all duration-300"
                          style={{ width: `${uploadProgress[fileObj.id] || 0}%` }}
                        />
                      </div>
                      <p className="text-xs text-gray-500 mt-1">
                        {uploadProgress[fileObj.id] || 0}% uploaded
                      </p>
                    </div>
                  )}
                </div>

                {/* Status & Actions */}
                <div className="flex items-center space-x-2">
                  {fileObj.status === 'pending' && (
                    <button
                      onClick={() => uploadFile(fileObj)}
                      className="text-blue-600 hover:text-blue-700 text-sm"
                    >
                      Upload
                    </button>
                  )}
                  
                  {fileObj.status === 'success' && (
                    <span className="text-green-600 text-sm">✓</span>
                  )}
                  
                  {fileObj.status === 'error' && (
                    <span className="text-red-600 text-sm">✗</span>
                  )}

                  <button
                    onClick={() => removeFile(fileObj.id)}
                    className="text-gray-400 hover:text-red-600"
                  >
                    <svg className="h-5 w-5" fill="currentColor" viewBox="0 0 20 20">
                      <path fillRule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clipRule="evenodd" />
                    </svg>
                  </button>
                </div>
              </div>
            ))}
          </div>
        </div>
      )}
    </div>
  );
}

// ✅ File upload API route
// pages/api/upload.js
import multer from 'multer';
import path from 'path';
import { promises as fs } from 'fs';

// ✅ Configure multer for file upload
const upload = multer({
  storage: multer.diskStorage({
    destination: async (req, file, cb) => {
      const uploadDir = path.join(process.cwd(), 'public/uploads');
      await fs.mkdir(uploadDir, { recursive: true });
      cb(null, uploadDir);
    },
    filename: (req, file, cb) => {
      const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
      const ext = path.extname(file.originalname);
      cb(null, file.fieldname + '-' + uniqueSuffix + ext);
    }
  }),
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB
  },
  fileFilter: (req, file, cb) => {
    // Allow only specific file types
    const allowedTypes = /jpeg|jpg|png|gif|pdf|doc|docx/;
    const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
    const mimetype = allowedTypes.test(file.mimetype);

    if (mimetype && extname) {
      return cb(null, true);
    } else {
      cb(new Error('Invalid file type'));
    }
  }
});

// ✅ Disable Next.js body parser for file uploads
export const config = {
  api: {
    bodyParser: false,
  },
};

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }

  try {
    await new Promise((resolve, reject) => {
      upload.single('file')(req, res, (err) => {
        if (err) {
          reject(err);
        } else {
          resolve();
        }
      });
    });

    if (!req.file) {
      return res.status(400).json({ message: 'No file uploaded' });
    }

    // ✅ Generate file URL
    const fileUrl = `/uploads/${req.file.filename}`;

    // ✅ Save file info to database (optional)
    // await prisma.file.create({
    //   data: {
    //     originalName: req.file.originalname,
    //     filename: req.file.filename,
    //     path: req.file.path,
    //     size: req.file.size,
    //     mimetype: req.file.mimetype,
    //     url: fileUrl
    //   }
    // });

    res.status(200).json({
      message: 'File uploaded successfully',
      file: {
        originalName: req.file.originalname,
        filename: req.file.filename,
        size: req.file.size,
        url: fileUrl
      }
    });
  } catch (error) {
    console.error('Upload error:', error);
    res.status(500).json({ 
      message: error.message || 'File upload failed' 
    });
  }
}
```

## ✅ Server-Side Validation

### **🛡️ API Validation**

```javascript
// lib/validation.js
import * as yup from 'yup';

// ✅ User registration schema
export const registrationSchema = yup.object({
  name: yup
    .string()
    .required('Name is required')
    .min(2, 'Name must be at least 2 characters')
    .max(50, 'Name must not exceed 50 characters')
    .matches(/^[a-zA-Z\s]+$/, 'Name can only contain letters and spaces'),
  
  email: yup
    .string()
    .required('Email is required')
    .email('Please enter a valid email address')
    .lowercase(),
  
  password: yup
    .string()
    .required('Password is required')
    .min(8, 'Password must be at least 8 characters')
    .matches(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/,
      'Password must contain at least one uppercase letter, one lowercase letter, one number, and one special character'
    ),
  
  confirmPassword: yup
    .string()
    .required('Please confirm your password')
    .oneOf([yup.ref('password')], 'Passwords must match'),
  
  dateOfBirth: yup
    .date()
    .required('Date of birth is required')
    .max(new Date(Date.now() - 13 * 365 * 24 * 60 * 60 * 1000), 'Must be at least 13 years old'),
  
  phone: yup
    .string()
    .matches(/^[+]?[(]?[\d\s\-().]+$/, 'Please enter a valid phone number')
    .optional(),
  
  terms: yup
    .boolean()
    .oneOf([true], 'You must accept the terms and conditions')
});

// ✅ Contact form schema
export const contactSchema = yup.object({
  name: yup
    .string()
    .required('Name is required')
    .min(2, 'Name must be at least 2 characters'),
  
  email: yup
    .string()
    .required('Email is required')
    .email('Please enter a valid email address'),
  
  subject: yup
    .string()
    .required('Subject is required')
    .min(5, 'Subject must be at least 5 characters'),
  
  message: yup
    .string()
    .required('Message is required')
    .min(10, 'Message must be at least 10 characters')
    .max(1000, 'Message must not exceed 1000 characters'),
  
  category: yup
    .string()
    .required('Please select a category')
    .oneOf(['general', 'support', 'sales', 'feedback'], 'Invalid category')
});

// ✅ Validation middleware
export function validateRequest(schema) {
  return async (req, res, next) => {
    try {
      // ✅ Validate request body
      const validatedData = await schema.validate(req.body, {
        abortEarly: false, // Return all validation errors
        stripUnknown: true // Remove unknown fields
      });

      // ✅ Replace request body with validated data
      req.body = validatedData;

      if (next) {
        return next();
      }
    } catch (error) {
      if (error.name === 'ValidationError') {
        const errors = {};
        
        error.inner.forEach(err => {
          errors[err.path] = err.message;
        });

        return res.status(400).json({
          message: 'Validation failed',
          errors
        });
      }

      return res.status(500).json({
        message: 'Validation error occurred'
      });
    }
  };
}

// ✅ Custom validation rules
export const customValidations = {
  // Check if email is already taken
  uniqueEmail: async (email) => {
    const existingUser = await prisma.user.findUnique({
      where: { email: email.toLowerCase() }
    });
    
    if (existingUser) {
      throw new yup.ValidationError(
        'This email is already registered',
        email,
        'email'
      );
    }
    
    return true;
  },

  // Validate file type and size
  validateFile: (file, allowedTypes = [], maxSize = 5 * 1024 * 1024) => {
    if (!file) {
      throw new yup.ValidationError('File is required', file, 'file');
    }

    if (allowedTypes.length > 0 && !allowedTypes.includes(file.mimetype)) {
      throw new yup.ValidationError(
        `File type not allowed. Allowed types: ${allowedTypes.join(', ')}`,
        file,
        'file'
      );
    }

    if (file.size > maxSize) {
      throw new yup.ValidationError(
        `File size too large. Maximum size: ${maxSize / (1024 * 1024)}MB`,
        file,
        'file'
      );
    }

    return true;
  },

  // Validate strong password
  strongPassword: (password) => {
    const checks = [
      { regex: /.{8,}/, message: 'At least 8 characters' },
      { regex: /[a-z]/, message: 'At least one lowercase letter' },
      { regex: /[A-Z]/, message: 'At least one uppercase letter' },
      { regex: /\d/, message: 'At least one number' },
      { regex: /[@$!%*?&]/, message: 'At least one special character' }
    ];

    const failedChecks = checks.filter(check => !check.regex.test(password));
    
    if (failedChecks.length > 0) {
      throw new yup.ValidationError(
        `Password must contain: ${failedChecks.map(c => c.message).join(', ')}`,
        password,
        'password'
      );
    }

    return true;
  }
};

// pages/api/contact.js
import { validateRequest, contactSchema } from '@/lib/validation';

async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }

  try {
    // ✅ Additional validation checks
    const { name, email, subject, message, category } = req.body;

    // ✅ Rate limiting check (example)
    const clientIP = req.headers['x-forwarded-for'] || req.connection.remoteAddress;
    const recentSubmissions = await prisma.contactSubmission.count({
      where: {
        ip: clientIP,
        createdAt: {
          gte: new Date(Date.now() - 60 * 60 * 1000) // Last hour
        }
      }
    });

    if (recentSubmissions >= 5) {
      return res.status(429).json({
        message: 'Too many submissions. Please try again later.'
      });
    }

    // ✅ Spam detection (basic example)
    const spamKeywords = ['viagra', 'casino', 'lottery', 'click here'];
    const hasSpam = spamKeywords.some(keyword => 
      message.toLowerCase().includes(keyword) || 
      subject.toLowerCase().includes(keyword)
    );

    if (hasSpam) {
      return res.status(400).json({
        message: 'Message appears to be spam'
      });
    }

    // ✅ Save to database
    const submission = await prisma.contactSubmission.create({
      data: {
        name,
        email,
        subject,
        message,
        category,
        ip: clientIP,
        userAgent: req.headers['user-agent']
      }
    });

    // ✅ Send email notification (example)
    // await sendEmail({
    //   to: process.env.ADMIN_EMAIL,
    //   subject: `New contact form submission: ${subject}`,
    //   template: 'contact-notification',
    //   data: { name, email, subject, message, category }
    // });

    res.status(200).json({
      message: 'Message sent successfully',
      id: submission.id
    });
  } catch (error) {
    console.error('Contact form error:', error);
    res.status(500).json({
      message: 'Failed to send message'
    });
  }
}

export default validateRequest(contactSchema)(handler);
```

## 📝 แบบฝึกหัด

### **🎯 แบบฝึกหัดที่ 1: Basic Form Implementation**
1. สร้าง contact form พร้อม validation
2. เพิ่ม client-side และ server-side validation
3. ทดสอบ error handling
4. เพิ่ม success/error feedback

### **🎯 แบบฝึกหัดที่ 2: React Hook Form**
1. ติดตั้งและ setup React Hook Form
2. สร้าง complex form พร้อม multiple field types
3. เพิ่ม dynamic validation
4. ทดสอบ form performance

### **🎯 แบบฝึกหัดที่ 3: File Upload System**
1. สร้าง file upload component
2. เพิ่ม drag & drop functionality
3. ติดตั้ง progress tracking
4. เพิ่ม file type และ size validation

### **🎯 แบบฝึกหัดที่ 4: Advanced Validation**
1. สร้าง custom validation rules
2. เพิ่ม async validation
3. ติดตั้ง rate limiting
4. เพิ่ม spam detection

## 🔗 การเชื่อมต่อ

**บทก่อนหน้า**: [บทที่ 11: Performance Optimization](./11-performance-optimization.md)

**บทต่อไป**: [บทที่ 13: App Router](./13-app-router.md)

**กลับไปหน้าหลัก**: [README](./README.md)

---

## 📚 เอกสารอ้างอิง

- [React Hook Form](https://react-hook-form.com/)
- [Yup Validation](https://github.com/jquense/yup)
- [React Dropzone](https://react-dropzone.js.org/)
- [Multer File Upload](https://github.com/expressjs/multer)
