class Auth0Provider {
    constructor(config) {
      this.domain = "dev-vfzstbs1dxkuagab.us.auth0.com";
      this.clientId = "CP6smmR7f8NVud69LolHjy8YWSPUZ7g0";
      this.redirectUri = config.redirectUri || window.location.origin;
      this.audience = '';
      if (config.audience && config.audience !== "This is unique one") {
        this.audience = config.audience;
      }
      
      this.scope = config.scope || ['openid', 'profile', 'email'];
      this.eventCallback = null;
      this.client = null;
    }
  
    onEvent(callback) {
      this.eventCallback = callback;
    }
  
    _emitEvent(event, data) {
      if (this.eventCallback) {
        this.eventCallback(event, data);
      }
    }
  
    async initialize() {
      try {
        await this._loadScript(`https://cdn.auth0.com/js/auth0-spa-js/2.0/auth0-spa-js.production.js`);
        
        this.client = await window.auth0.createAuth0Client({
          domain: this.domain,
          clientId: this.clientId,
          authorizationParams: {
            redirect_uri: this.redirectUri,
            audience: this.audience,
            scope: Array.isArray(this.scope) ? this.scope.join(' ') : this.scope
          }
        });
        
        if (window.location.search.includes('code=') && 
            window.location.search.includes('state=')) {
          await this._handleRedirectCallback();
        }
        
        const isAuthenticated = await this.client.isAuthenticated();
        
        if (isAuthenticated) {
          const user = await this.client.getUser();
          this._emitEvent('initialized', { authenticated: true, user });
        } else {
          this._emitEvent('initialized', { authenticated: false });
        }
        
        return this;
      } catch (error) {
        this._emitEvent('error', { error });
        throw error;
      }
    }
  
    _loadScript(src) {
      return new Promise((resolve, reject) => {
        if (document.querySelector(`script[src="${src}"]`)) {
          resolve();
          return;
        }
        
        const script = document.createElement('script');
        script.src = src;
        script.async = true;
        
        script.onload = () => resolve();
        script.onerror = (error) => reject(new Error(`Failed to load script: ${src}`));
        
        document.body.appendChild(script);
      });
    }
  
    async _handleRedirectCallback() {
      try {
        const result = await this.client.handleRedirectCallback();
        const user = await this.client.getUser();
        
        window.history.replaceState({}, document.title, window.location.pathname);
        
        this._emitEvent('authenticated', { user });
        
        return result;
      } catch (error) {
        this._emitEvent('error', { error });
        throw error;
      }
    }
  
    async login(options = {}) {
      try {
        await this.client.loginWithRedirect({
          authorizationParams: {
            ...options
          }
        });
        
        return true;
      } catch (error) {
        this._emitEvent('error', { error });
        throw error;
      }
    }
  
    async loginWithPopup(options = {}) {
      try {
        await this.client.loginWithPopup({
          authorizationParams: {
            ...options
          }
        });
        
        const user = await this.client.getUser();
        this._emitEvent('authenticated', { user });
        
        return user;
      } catch (error) {
        this._emitEvent('error', { error });
        throw error;
      }
    }
  
    async logout(options = {}) {
      try {
        await this.client.logout({
          logoutParams: {
            returnTo: options.returnTo || window.location.origin
          }
        });
        
        this._emitEvent('logout', {});
        
        return true;
      } catch (error) {
        this._emitEvent('error', { error });
        throw error;
      }
    }
  
    async getAccessToken() {
      try {
        const token = await this.client.getTokenSilently();
        return token;
      } catch (error) {
        this._emitEvent('error', { error });
        throw error;
      }
    }
  
    async isAuthenticated() {
      try {
        return await this.client.isAuthenticated();
      } catch (error) {
        this._emitEvent('error', { error });
        return false;
      }
    }
  
    async getUserProfile() {
      try {
        return await this.client.getUser();
      } catch (error) {
        this._emitEvent('error', { error });
        throw error;
      }
    }
  }
  
  export { Auth0Provider };
