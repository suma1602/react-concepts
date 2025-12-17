# React Forms: Comprehensive Guide

## Table of Contents
1. [Controlled Components](#controlled-components)
2. [Uncontrolled Components](#uncontrolled-components)
3. [Form Validation](#form-validation)
4. [Form Libraries](#form-libraries)
5. [Advanced Patterns](#advanced-patterns)
6. [Best Practices](#best-practices)

---

## Controlled Components

Controlled components are form elements whose values are managed by React state. The component state is the "single source of truth" for the form data.

### Basic Example

```jsx
import React, { useState } from 'react';

function ControlledForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prevState => ({
      ...prevState,
      [name]: value
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Enter your name"
      />
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Enter your email"
      />
      <textarea
        name="message"
        value={formData.message}
        onChange={handleChange}
        placeholder="Enter your message"
      />
      <button type="submit">Submit</button>
    </form>
  );
}

export default ControlledForm;
```

### Advantages
- **Full control**: Complete access to form state at all times
- **Validation on change**: Can validate and provide real-time feedback
- **Conditional rendering**: Can show/hide fields based on other values
- **Easy reset**: Simple to reset all fields to initial state
- **Instant value synchronization**: State always matches input values

### Disadvantages
- **More boilerplate**: Requires more code for setup
- **Performance**: Extra re-renders as state updates for each keystroke
- **Complexity**: Can become complex with large forms

### Advanced Controlled Components

```jsx
import React, { useState } from 'react';

function AdvancedControlledForm() {
  const [formData, setFormData] = useState({
    username: '',
    password: '',
    confirmPassword: '',
    agreeToTerms: false,
    country: 'US'
  });

  const [touched, setTouched] = useState({});

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prevState => ({
      ...prevState,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prevState => ({
      ...prevState,
      [name]: true
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form data:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Username:</label>
        <input
          type="text"
          name="username"
          value={formData.username}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.username && formData.username.length < 3 && (
          <error>Username must be at least 3 characters</error>
        )}
      </div>

      <div>
        <label>Password:</label>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
          onBlur={handleBlur}
        />
      </div>

      <div>
        <label>Country:</label>
        <select
          name="country"
          value={formData.country}
          onChange={handleChange}
        >
          <option value="US">United States</option>
          <option value="UK">United Kingdom</option>
          <option value="CA">Canada</option>
        </select>
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            name="agreeToTerms"
            checked={formData.agreeToTerms}
            onChange={handleChange}
          />
          I agree to the terms
        </label>
      </div>

      <button type="submit" disabled={!formData.agreeToTerms}>
        Submit
      </button>
    </form>
  );
}

export default AdvancedControlledForm;
```

---

## Uncontrolled Components

Uncontrolled components manage their own state internally, similar to traditional HTML forms. React doesn't control the form data; the DOM does.

### Basic Example

```jsx
import React, { useRef } from 'react';

function UncontrolledForm() {
  const nameRef = useRef(null);
  const emailRef = useRef(null);
  const messageRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    const formData = {
      name: nameRef.current.value,
      email: emailRef.current.value,
      message: messageRef.current.value
    };
    console.log('Form submitted:', formData);
  };

  const handleReset = () => {
    nameRef.current.value = '';
    emailRef.current.value = '';
    messageRef.current.value = '';
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        ref={nameRef}
        placeholder="Enter your name"
        defaultValue="John Doe"
      />
      <input
        type="email"
        ref={emailRef}
        placeholder="Enter your email"
      />
      <textarea
        ref={messageRef}
        placeholder="Enter your message"
      />
      <button type="submit">Submit</button>
      <button type="button" onClick={handleReset}>Reset</button>
    </form>
  );
}

export default UncontrolledForm;
```

### Advantages
- **Less code**: Simpler implementation for simple forms
- **Better performance**: Fewer re-renders
- **File inputs**: Native way to handle file uploads
- **Integration**: Easy integration with non-React code

### Disadvantages
- **No real-time validation**: Can't validate as user types
- **Harder to disable submit**: Must access DOM to check values
- **No conditional rendering**: Can't show/hide fields dynamically
- **Testing complexity**: Harder to test

### When to Use Uncontrolled Components

```jsx
// File input - uncontrolled is often better
function FileUpload() {
  const fileRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    const file = fileRef.current.files[0];
    console.log('Selected file:', file);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="file" ref={fileRef} />
      <button type="submit">Upload</button>
    </form>
  );
}
```

---

## Form Validation

### Real-time Validation with Controlled Components

```jsx
import React, { useState } from 'react';

function FormWithValidation() {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    confirmPassword: ''
  });

  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const validateEmail = (email) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  };

  const validatePassword = (password) => {
    return password.length >= 8;
  };

  const validateForm = (data) => {
    const newErrors = {};

    if (!validateEmail(data.email)) {
      newErrors.email = 'Invalid email format';
    }

    if (!validatePassword(data.password)) {
      newErrors.password = 'Password must be at least 8 characters';
    }

    if (data.password !== data.confirmPassword) {
      newErrors.confirmPassword = 'Passwords do not match';
    }

    return newErrors;
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prevState => ({
      ...prevState,
      [name]: value
    }));

    // Real-time validation for touched fields
    if (touched[name]) {
      const newErrors = validateForm({ ...formData, [name]: value });
      setErrors(prevErrors => ({
        ...prevErrors,
        [name]: newErrors[name]
      }));
    }
  };

  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prevState => ({
      ...prevState,
      [name]: true
    }));

    // Validate on blur
    const newErrors = validateForm(formData);
    setErrors(prevErrors => ({
      ...prevErrors,
      [name]: newErrors[name]
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validateForm(formData);

    if (Object.keys(newErrors).length === 0) {
      console.log('Form is valid, submitting:', formData);
    } else {
      setErrors(newErrors);
      setTouched({
        email: true,
        password: true,
        confirmPassword: true
      });
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div className="form-group">
        <label>Email:</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          onBlur={handleBlur}
          className={errors.email ? 'input-error' : ''}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>

      <div className="form-group">
        <label>Password:</label>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
          onBlur={handleBlur}
          className={errors.password ? 'input-error' : ''}
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>

      <div className="form-group">
        <label>Confirm Password:</label>
        <input
          type="password"
          name="confirmPassword"
          value={formData.confirmPassword}
          onChange={handleChange}
          onBlur={handleBlur}
          className={errors.confirmPassword ? 'input-error' : ''}
        />
        {errors.confirmPassword && <span className="error">{errors.confirmPassword}</span>}
      </div>

      <button
        type="submit"
        disabled={Object.keys(errors).length > 0}
      >
        Submit
      </button>
    </form>
  );
}

export default FormWithValidation;
```

### Async Validation

```jsx
import React, { useState } from 'react';

function FormWithAsyncValidation() {
  const [username, setUsername] = useState('');
  const [error, setError] = useState('');
  const [isChecking, setIsChecking] = useState(false);

  const checkUsernameAvailability = async (username) => {
    if (username.length < 3) {
      setError('Username must be at least 3 characters');
      return;
    }

    setIsChecking(true);
    setError('');

    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      const takenUsernames = ['admin', 'user', 'test'];
      if (takenUsernames.includes(username.toLowerCase())) {
        setError('Username is already taken');
      } else {
        setError('');
      }
    } catch (err) {
      setError('Error checking availability');
    } finally {
      setIsChecking(false);
    }
  };

  const handleChange = (e) => {
    setUsername(e.target.value);
  };

  const handleBlur = () => {
    checkUsernameAvailability(username);
  };

  return (
    <div>
      <input
        type="text"
        value={username}
        onChange={handleChange}
        onBlur={handleBlur}
        placeholder="Enter username"
      />
      {isChecking && <span>Checking availability...</span>}
      {error && <span className="error">{error}</span>}
    </div>
  );
}

export default FormWithAsyncValidation;
```

---

## Form Libraries

### React Hook Form

React Hook Form provides excellent performance and minimal re-renders.

```jsx
import React from 'react';
import { useForm } from 'react-hook-form';

function FormWithHookForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
    watch,
    reset
  } = useForm({
    defaultValues: {
      email: '',
      password: '',
      agreeToTerms: false
    }
  });

  const password = watch('password');

  const onSubmit = (data) => {
    console.log('Form data:', data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Email:</label>
        <input
          {...register('email', {
            required: 'Email is required',
            pattern: {
              value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
              message: 'Invalid email format'
            }
          })}
        />
        {errors.email && <span className="error">{errors.email.message}</span>}
      </div>

      <div>
        <label>Password:</label>
        <input
          type="password"
          {...register('password', {
            required: 'Password is required',
            minLength: {
              value: 8,
              message: 'Password must be at least 8 characters'
            }
          })}
        />
        {errors.password && <span className="error">{errors.password.message}</span>}
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            {...register('agreeToTerms', {
              required: 'You must agree to terms'
            })}
          />
          I agree to the terms
        </label>
        {errors.agreeToTerms && <span className="error">{errors.agreeToTerms.message}</span>}
      </div>

      <button type="submit">Submit</button>
      <button type="button" onClick={() => reset()}>Reset</button>
    </form>
  );
}

export default FormWithHookForm;
```

**Advantages:**
- Minimal re-renders
- Great performance
- Small bundle size
- Extensive validation rules
- Good TypeScript support

### Formik

Formik is a popular form management library with built-in validation.

```jsx
import React from 'react';
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';

const validationSchema = Yup.object({
  email: Yup.string()
    .email('Invalid email')
    .required('Email is required'),
  password: Yup.string()
    .min(8, 'Password must be at least 8 characters')
    .required('Password is required'),
  confirmPassword: Yup.string()
    .oneOf([Yup.ref('password')], 'Passwords must match')
    .required('Confirm password is required')
});

function FormWithFormik() {
  return (
    <Formik
      initialValues={{
        email: '',
        password: '',
        confirmPassword: ''
      }}
      validationSchema={validationSchema}
      onSubmit={(values) => {
        console.log('Form submitted:', values);
      }}
    >
      {({ isSubmitting, isValid }) => (
        <Form>
          <div>
            <label>Email:</label>
            <Field
              type="email"
              name="email"
              placeholder="Enter your email"
            />
            <ErrorMessage name="email" component="span" className="error" />
          </div>

          <div>
            <label>Password:</label>
            <Field
              type="password"
              name="password"
              placeholder="Enter your password"
            />
            <ErrorMessage name="password" component="span" className="error" />
          </div>

          <div>
            <label>Confirm Password:</label>
            <Field
              type="password"
              name="confirmPassword"
              placeholder="Confirm your password"
            />
            <ErrorMessage name="confirmPassword" component="span" className="error" />
          </div>

          <button
            type="submit"
            disabled={!isValid || isSubmitting}
          >
            {isSubmitting ? 'Submitting...' : 'Submit'}
          </button>
        </Form>
      )}
    </Formik>
  );
}

export default FormWithFormik;
```

**Advantages:**
- Comprehensive validation with Yup
- Good component integration
- Excellent documentation
- Built-in error handling
- Field-level validation

### Comparison Table

| Feature | Controlled | React Hook Form | Formik |
|---------|-----------|-----------------|--------|
| Re-renders | Higher | Very Low | Low |
| Bundle Size | N/A | Small | Medium |
| Learning Curve | Easy | Medium | Medium |
| Validation | Manual | Built-in | Built-in |
| Performance | Good | Excellent | Good |
| TypeScript Support | Good | Excellent | Good |

---

## Advanced Patterns

### Dynamic Form Fields

```jsx
import React, { useState } from 'react';

function DynamicForm() {
  const [fields, setFields] = useState([{ id: 1, value: '' }]);
  const [nextId, setNextId] = useState(2);

  const addField = () => {
    setFields([...fields, { id: nextId, value: '' }]);
    setNextId(nextId + 1);
  };

  const removeField = (id) => {
    setFields(fields.filter(field => field.id !== id));
  };

  const updateField = (id, value) => {
    setFields(fields.map(field =>
      field.id === id ? { ...field, value } : field
    ));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form data:', fields);
  };

  return (
    <form onSubmit={handleSubmit}>
      {fields.map((field) => (
        <div key={field.id} className="field-group">
          <input
            type="text"
            value={field.value}
            onChange={(e) => updateField(field.id, e.target.value)}
            placeholder={`Field ${field.id}`}
          />
          <button
            type="button"
            onClick={() => removeField(field.id)}
            disabled={fields.length === 1}
          >
            Remove
          </button>
        </div>
      ))}
      <button type="button" onClick={addField}>Add Field</button>
      <button type="submit">Submit</button>
    </form>
  );
}

export default DynamicForm;
```

### Multi-Step Form

```jsx
import React, { useState } from 'react';

function MultiStepForm() {
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    firstName: '',
    lastName: ''
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };

  const handleNext = () => {
    if (step < 3) setStep(step + 1);
  };

  const handlePrevious = () => {
    if (step > 1) setStep(step - 1);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Final form data:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div className="progress">Step {step} of 3</div>

      {step === 1 && (
        <div>
          <h2>Step 1: Account Details</h2>
          <input
            type="email"
            name="email"
            value={formData.email}
            onChange={handleChange}
            placeholder="Email"
          />
          <input
            type="password"
            name="password"
            value={formData.password}
            onChange={handleChange}
            placeholder="Password"
          />
        </div>
      )}

      {step === 2 && (
        <div>
          <h2>Step 2: Personal Information</h2>
          <input
            type="text"
            name="firstName"
            value={formData.firstName}
            onChange={handleChange}
            placeholder="First Name"
          />
          <input
            type="text"
            name="lastName"
            value={formData.lastName}
            onChange={handleChange}
            placeholder="Last Name"
          />
        </div>
      )}

      {step === 3 && (
        <div>
          <h2>Step 3: Review</h2>
          <p>Email: {formData.email}</p>
          <p>Name: {formData.firstName} {formData.lastName}</p>
        </div>
      )}

      <div className="buttons">
        {step > 1 && <button type="button" onClick={handlePrevious}>Previous</button>}
        {step < 3 && <button type="button" onClick={handleNext}>Next</button>}
        {step === 3 && <button type="submit">Submit</button>}
      </div>
    </form>
  );
}

export default MultiStepForm;
```

### Form State Management with useReducer

```jsx
import React, { useReducer } from 'react';

const initialState = {
  email: '',
  password: '',
  confirmPassword: '',
  errors: {}
};

function formReducer(state, action) {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        [action.payload.name]: action.payload.value
      };
    case 'SET_ERROR':
      return {
        ...state,
        errors: {
          ...state.errors,
          [action.payload.field]: action.payload.error
        }
      };
    case 'CLEAR_ERRORS':
      return {
        ...state,
        errors: {}
      };
    case 'RESET':
      return initialState;
    default:
      return state;
  }
}

function FormWithReducer() {
  const [state, dispatch] = useReducer(formReducer, initialState);

  const handleChange = (e) => {
    const { name, value } = e.target;
    dispatch({
      type: 'SET_FIELD',
      payload: { name, value }
    });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = {};

    if (!state.email) {
      newErrors.email = 'Email is required';
    }

    if (state.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }

    if (state.password !== state.confirmPassword) {
      newErrors.confirmPassword = 'Passwords do not match';
    }

    if (Object.keys(newErrors).length > 0) {
      Object.entries(newErrors).forEach(([field, error]) => {
        dispatch({
          type: 'SET_ERROR',
          payload: { field, error }
        });
      });
    } else {
      console.log('Form submitted:', state);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          name="email"
          value={state.email}
          onChange={handleChange}
          placeholder="Email"
        />
        {state.errors.email && <span className="error">{state.errors.email}</span>}
      </div>

      <div>
        <input
          type="password"
          name="password"
          value={state.password}
          onChange={handleChange}
          placeholder="Password"
        />
        {state.errors.password && <span className="error">{state.errors.password}</span>}
      </div>

      <div>
        <input
          type="password"
          name="confirmPassword"
          value={state.confirmPassword}
          onChange={handleChange}
          placeholder="Confirm Password"
        />
        {state.errors.confirmPassword && <span className="error">{state.errors.confirmPassword}</span>}
      </div>

      <button type="submit">Submit</button>
      <button type="button" onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </form>
  );
}

export default FormWithReducer;
```

### File Upload Handler

```jsx
import React, { useState } from 'react';

function FileUploadForm() {
  const [file, setFile] = useState(null);
  const [preview, setPreview] = useState(null);
  const [uploadProgress, setUploadProgress] = useState(0);
  const [isUploading, setIsUploading] = useState(false);

  const handleFileChange = (e) => {
    const selectedFile = e.target.files[0];
    
    if (selectedFile) {
      setFile(selectedFile);

      // Create preview for images
      if (selectedFile.type.startsWith('image/')) {
        const reader = new FileReader();
        reader.onload = (event) => {
          setPreview(event.target.result);
        };
        reader.readAsDataURL(selectedFile);
      }
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    if (!file) {
      alert('Please select a file');
      return;
    }

    setIsUploading(true);

    const formData = new FormData();
    formData.append('file', file);

    try {
      // Simulate file upload with progress
      await simulateUpload();
      console.log('File uploaded successfully');
      setFile(null);
      setPreview(null);
      setUploadProgress(0);
    } catch (error) {
      console.error('Upload failed:', error);
    } finally {
      setIsUploading(false);
    }
  };

  const simulateUpload = () => {
    return new Promise((resolve) => {
      let progress = 0;
      const interval = setInterval(() => {
        progress += Math.random() * 30;
        if (progress >= 100) {
          progress = 100;
          clearInterval(interval);
          setUploadProgress(progress);
          resolve();
        } else {
          setUploadProgress(progress);
        }
      }, 500);
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Upload File:</label>
        <input
          type="file"
          onChange={handleFileChange}
          accept="image/*,.pdf"
          disabled={isUploading}
        />
      </div>

      {preview && (
        <div>
          <img src={preview} alt="Preview" style={{ maxWidth: '200px' }} />
        </div>
      )}

      {file && (
        <div>
          <p>Selected file: {file.name}</p>
          <p>Size: {(file.size / 1024 / 1024).toFixed(2)} MB</p>
        </div>
      )}

      {isUploading && (
        <div>
          <progress value={uploadProgress} max="100" />
          <p>{Math.round(uploadProgress)}%</p>
        </div>
      )}

      <button type="submit" disabled={!file || isUploading}>
        {isUploading ? 'Uploading...' : 'Upload'}
      </button>
    </form>
  );
}

export default FileUploadForm;
```

---

## Best Practices

### 1. Choose the Right Approach

```jsx
// Use controlled components when:
// - You need real-time validation
// - You need to conditionally render fields
// - You need to sync state across multiple fields

// Use uncontrolled components when:
// - Dealing with file inputs
// - Integrating with non-React code
// - Building very simple forms
```

### 2. Separate Concerns

```jsx
// Create custom hooks for form logic
function useForm(initialValues, onSubmit) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: value
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(values);
  };

  return { values, errors, handleChange, handleSubmit, setErrors };
}

// Usage
function MyForm() {
  const { values, handleChange, handleSubmit } = useForm(
    { email: '', password: '' },
    (values) => console.log('Submitted:', values)
  );

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" value={values.email} onChange={handleChange} />
      <input name="password" type="password" value={values.password} onChange={handleChange} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### 3. Debounce Validation

```jsx
import React, { useState, useEffect, useRef } from 'react';

function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

function FormWithDebounce() {
  const [email, setEmail] = useState('');
  const [isValidating, setIsValidating] = useState(false);
  const [isValid, setIsValid] = useState(false);

  const debouncedEmail = useDebounce(email, 500);

  useEffect(() => {
    if (debouncedEmail) {
      setIsValidating(true);
      // Simulate API call
      setTimeout(() => {
        setIsValid(/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(debouncedEmail));
        setIsValidating(false);
      }, 500);
    }
  }, [debouncedEmail]);

  return (
    <div>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Enter email"
      />
      {isValidating && <span>Checking...</span>}
      {!isValidating && isValid && <span className="success">✓ Valid</span>}
      {!isValidating && email && !isValid && <span className="error">✗ Invalid</span>}
    </div>
  );
}

export default FormWithDebounce;
```

### 4. Accessibility Considerations

```jsx
function AccessibleForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});
  const emailErrorId = 'email-error';
  const passwordErrorId = 'password-error';

  const handleSubmit = (e) => {
    e.preventDefault();
    // Validation logic
  };

  return (
    <form onSubmit={handleSubmit} noValidate>
      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? emailErrorId : undefined}
          required
        />
        {errors.email && (
          <span id={emailErrorId} role="alert" className="error">
            {errors.email}
          </span>
        )}
      </div>

      <div>
        <label htmlFor="password">Password:</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          aria-invalid={!!errors.password}
          aria-describedby={errors.password ? passwordErrorId : undefined}
          required
        />
        {errors.password && (
          <span id={passwordErrorId} role="alert" className="error">
            {errors.password}
          </span>
        )}
      </div>

      <button type="submit">Submit</button>
    </form>
  );
}

export default AccessibleForm;
```

### 5. Performance Optimization

```jsx
import React, { useState, useCallback, useMemo } from 'react';

function OptimizedForm() {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });

  // Memoize the validation function
  const validator = useMemo(() => {
    return {
      email: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
      password: (value) => value.length >= 8
    };
  }, []);

  // Memoize the handler to prevent unnecessary re-renders of child components
  const handleChange = useCallback((e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  }, []);

  const isFormValid = useMemo(() => {
    return validator.email(formData.email) && validator.password(formData.password);
  }, [formData, validator]);

  return (
    <form>
      <input
        name="email"
        value={formData.email}
        onChange={handleChange}
      />
      <input
        name="password"
        type="password"
        value={formData.password}
        onChange={handleChange}
      />
      <button disabled={!isFormValid}>Submit</button>
    </form>
  );
}

export default OptimizedForm;
```

### 6. Error Handling Best Practices

```jsx
function FormWithErrorBoundary() {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });

  const [formState, setFormState] = useState({
    isLoading: false,
    error: null,
    success: false
  });

  const handleSubmit = async (e) => {
    e.preventDefault();
    setFormState({ isLoading: true, error: null, success: false });

    try {
      // Simulate API call
      const response = await fetch('/api/form', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });

      if (!response.ok) {
        throw new Error('Form submission failed');
      }

      setFormState({ isLoading: false, error: null, success: true });
    } catch (error) {
      setFormState({
        isLoading: false,
        error: error.message,
        success: false
      });
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {formState.error && (
        <div role="alert" className="error-message">
          {formState.error}
        </div>
      )}

      {formState.success && (
        <div role="status" className="success-message">
          Form submitted successfully!
        </div>
      )}

      <input
        type="email"
        value={formData.email}
        onChange={(e) => setFormData(prev => ({ ...prev, email: e.target.value }))}
        disabled={formState.isLoading}
      />

      <button type="submit" disabled={formState.isLoading}>
        {formState.isLoading ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}

export default FormWithErrorBoundary;
```

### 7. Testing Forms

```jsx
// test.jsx - Example with React Testing Library
import { render, screen, userEvent } from '@testing-library/react';
import MyForm from './MyForm';

describe('MyForm', () => {
  it('should display validation error on invalid email', async () => {
    render(<MyForm />);
    
    const emailInput = screen.getByLabelText('Email');
    const submitButton = screen.getByRole('button', { name: /submit/i });

    await userEvent.type(emailInput, 'invalid-email');
    await userEvent.click(submitButton);

    expect(screen.getByText('Invalid email format')).toBeInTheDocument();
  });

  it('should submit form with valid data', async () => {
    render(<MyForm />);

    const emailInput = screen.getByLabelText('Email');
    const passwordInput = screen.getByLabelText('Password');
    const submitButton = screen.getByRole('button', { name: /submit/i });

    await userEvent.type(emailInput, 'test@example.com');
    await userEvent.type(passwordInput, 'password123');
    await userEvent.click(submitButton);

    // Assert successful submission
  });
});
```

---

## Summary

### Controlled vs Uncontrolled

| Aspect | Controlled | Uncontrolled |
|--------|-----------|-------------|
| State Management | React state | DOM |
| Real-time Validation | ✅ Easy | ❌ Difficult |
| Dynamic Fields | ✅ Easy | ❌ Hard |
| Performance | ⚠️ Moderate | ✅ Better |
| File Inputs | ⚠️ Limited | ✅ Native |
| Testing | ✅ Easier | ⚠️ Harder |

### Library Recommendations

- **Small forms**: Controlled components or React Hook Form
- **Complex validation**: Formik or React Hook Form
- **Large forms**: React Hook Form (better performance)
- **File uploads**: Uncontrolled components
- **Enterprise**: React Hook Form + custom validation

### Key Takeaways

1. Choose the right approach for your use case
2. Keep validation logic separate and reusable
3. Always provide clear error messages
4. Consider accessibility from the start
5. Use libraries for complex forms
6. Test form interactions thoroughly
7. Optimize for performance with large forms
8. Handle errors gracefully
