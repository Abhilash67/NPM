import React, { createContext, useContext, useState, useEffect } from 'react';
import { AuthProvider } from './AuthProvider';

// Create context
const AuthContext = createContext(null);

export const AuthContextProvider = ({ 
  children, 
  providerType = 'auth0',
  providerConfig = {},
  onLogin,
  onLogout,
  onError
}) => {
  const [authProvider, setAuthProvider] = useState(null);
  const [user, setUser] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const initAuth = async () => {
      try {
        setIsLoading(true);
        setError(null);
        
        const provider = new AuthProvider({
          providerType,
          providerConfig
        });
        
        provider.on('initialized', (data) => {
          setIsAuthenticated(data.authenticated);
          if (data.user) {
            setUser(data.user);
          }
          setIsLoading(false);
        });
        
        provider.on('authenticated', (data) => {
          setIsAuthenticated(true);
          setUser(data.user);
          if (onLogin) {
            onLogin(data.user);
          }
        });
        
        provider.on('logout', () => {
          setIsAuthenticated(false);
          setUser(null);
          if (onLogout) {
            onLogout();
          }
        });
        
        provider.on('error', (data) => {
          setError(data.error);
          if (onError) {
            onError(data.error);
          }
        });
        
        await provider.initialize();
        
        setAuthProvider(provider);
      } catch (err) {
        setError(err);
        setIsLoading(false);
        if (onError) {
          onError(err);
        }
      }
    };

    initAuth();
    
    return () => {};
  }, [providerType, providerConfig, onLogin, onLogout, onError]);

  const login = async (options) => {
    if (!authProvider) {
      throw new Error('Auth provider not initialized');
    }
    
    setIsLoading(true);
    try {
      await authProvider.login(options);
    } catch (err) {
      setError(err);
      setIsLoading(false);
      throw err;
    }
  };

  const loginWithPopup = async (options) => {
    if (!authProvider) {
      throw new Error('Auth provider not initialized');
    }
    
    setIsLoading(true);
    try {
      const user = await authProvider.loginWithPopup(options);
      setIsLoading(false);
      return user;
    } catch (err) {
      setError(err);
      setIsLoading(false);
      throw err;
    }
  };

  const logout = async (options) => {
    if (!authProvider) {
      throw new Error('Auth provider not initialized');
    }
    
    try {
      await authProvider.logout(options);
    } catch (err) {
      setError(err);
      throw err;
    }
  };

  const getAccessToken = async () => {
    if (!authProvider) {
      throw new Error('Auth provider not initialized');
    }
    
    try {
      return await authProvider.getAccessToken();
    } catch (err) {
      setError(err);
      throw err;
    }
  };

  const contextValue = {
    isAuthenticated,
    isLoading,
    user,
    error,
    login,
    loginWithPopup,
    logout,
    getAccessToken
  };

  return (
    <AuthContext.Provider value={contextValue}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthContextProvider');
  }
  return context;
};

export const withAuth = (Component) => {
  const WithAuth = (props) => {
    const auth = useAuth();
    
    if (auth.isLoading) {
      return <div>Loading...</div>;
    }
    
    if (!auth.isAuthenticated) {
      return <div>Please log in to view this page</div>;
    }
    
    return <Component {...props} auth={auth} />;
  };
  
  return WithAuth;
};
