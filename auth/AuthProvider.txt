import { Auth0Provider } from './Providers/Auth0Provider';

class AuthProvider {
  constructor(config = {}) {
    this.providerType = config.providerType || 'auth0';
    this.providerConfig = {
      ...config.providerConfig
    };
    this.provider = null;
    this.initialized = false;
    this.eventListeners = {};
    
    this._initProvider();
  }

  _initProvider() {
    // Factory pattern for supporting multiple providers
    switch (this.providerType) {
      case 'auth0':
      default:
        this.provider = new Auth0Provider(this.providerConfig);
        break;
      // Add other providers here in the future
      // case 'okta':
      //   this.provider = new OktaProvider(this.providerConfig);
      //   break;
    }
    
    this.provider.onEvent((event, data) => {
      this._dispatchEvent(event, data);
    });
    
    this.initialized = true;
  }

  async initialize() {
    if (!this.provider) {
      throw new Error('Provider not initialized');
    }
    
    await this.provider.initialize();
    return this;
  }

  async login(options = {}) {
    if (!this.provider) {
      throw new Error('Provider not initialized');
    }
    
    return this.provider.login(options);
  }

  async loginWithPopup(options = {}) {
    if (!this.provider) {
      throw new Error('Provider not initialized');
    }
    
    return this.provider.loginWithPopup(options);
  }

  async logout(options = {}) {
    if (!this.provider) {
      throw new Error('Provider not initialized');
    }
    
    return this.provider.logout(options);
  }

  async getAccessToken() {
    if (!this.provider) {
      throw new Error('Provider not initialized');
    }
    
    return this.provider.getAccessToken();
  }

  async isAuthenticated() {
    if (!this.provider) {
      throw new Error('Provider not initialized');
    }
    
    return this.provider.isAuthenticated();
  }

  async getUserProfile() {
    if (!this.provider) {
      throw new Error('Provider not initialized');
    }
    
    return this.provider.getUserProfile();
  }

  on(event, callback) {
    if (!this.eventListeners[event]) {
      this.eventListeners[event] = [];
    }
    
    this.eventListeners[event].push(callback);
  }

  off(event, callback) {
    if (!this.eventListeners[event]) {
      return;
    }
    
    this.eventListeners[event] = this.eventListeners[event].filter(
      cb => cb !== callback
    );
  }

  _dispatchEvent(event, data) {
    if (!this.eventListeners[event]) {
      return;
    }
    
    for (const callback of this.eventListeners[event]) {
      callback(data);
    }
  }
}

export { AuthProvider };
