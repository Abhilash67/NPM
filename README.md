# Auth Provider SDK

A provider-agnostic authentication SDK that serves as a bridge between your application and authentication providers like Auth0. This package allows you to easily switch authentication providers in the future without changing your application code.

## Installation

```bash
npm install auth-provider-sdk
# or
yarn add auth-provider-sdk
```

## Usage

1. Wrap your application with `AuthContextProvider`:

```jsx
// App.js
import React from 'react';
import { AuthContextProvider } from 'auth-provider-sdk';

function App() {
  const providerConfig = {
    domain: 'your-domain.auth0.com',
    clientId: 'your-client-id',
    audience: 'your-audience',
    redirectUri: window.location.origin
  };

  return (
    <AuthContextProvider
      providerType="auth0"
      providerConfig={providerConfig}
      onError={(error) => console.error("Auth error:", error)}
    >
      <YourApp />
    </AuthContextProvider>
  );
}
```

2. Use the authentication hooks in your components:

```jsx
// LoginPage.js
import React from 'react';
import { useAuth } from 'auth-provider-sdk';

function LoginPage() {
  const { isAuthenticated, user, login, logout, isLoading } = useAuth();

  return (
    <div>
      <h1>Authentication</h1>
      
      {isLoading ? (
        <div>Loading...</div>
      ) : isAuthenticated ? (
        <div>
          <p>Hello, {user.name}!</p>
          <button onClick={() => logout()}>Log Out</button>
        </div>
      ) : (
        <div>
          <p>Please log in</p>
          <button onClick={() => login()}>Log In</button>
          <button onClick={() => login({ connection: 'google-oauth2' })}>
            Log In with Google
          </button>
        </div>
      )}
    </div>
  );
}
```

## API Reference

### Components

- `AuthContextProvider` - Main provider component for authentication

### Hooks

- `useAuth()` - Hook to access authentication context with these properties:
  - `isAuthenticated` - Boolean indicating if user is authenticated
  - `isLoading` - Boolean indicating if authentication is loading
  - `user` - User profile when authenticated
  - `error` - Error object if authentication fails
  - `login(options)` - Function to trigger login redirect
  - `loginWithPopup(options)` - Function to trigger login in popup
  - `logout(options)` - Function to log out
  - `getAccessToken()` - Function to get access token

### HOCs

- `withAuth(Component)` - HOC for class components

## Configuration

The SDK can be configured with the following options:

```jsx
<AuthContextProvider
  // The authentication provider to use ('auth0', etc.)
  providerType="auth0"
  
  // Provider-specific configuration
  providerConfig={{
    domain: 'your-domain.auth0.com',
    clientId: 'your-client-id',
    audience: 'your-audience',
    redirectUri: window.location.origin
  }}
  
  // Event callbacks
  onLogin={(user) => console.log('User logged in', user)}
  onLogout={() => console.log('User logged out')}
  onError={(error) => console.error('Auth error', error)}
>
  {/* Your app */}
</AuthContextProvider>
```

## License

MIT
