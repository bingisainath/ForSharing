
shatrughan.teamlease@avanse.com


import React from 'react';
import { useNavigate } from 'react-router-dom';
import './Splash.css';

const Splash = () => {
  const navigate = useNavigate();

  const handleGetStarted = () => {
    navigate('/login'); // Navigate to login page
  };

  return (
    <div className="splash-container">
      <div className="splash-content">
        <h1>Welcome to Our App</h1>
        <p>Your journey to a better experience starts here. Explore the features and enjoy seamless integration with your favorite tools.</p>
        <button className="get-started-button" onClick={handleGetStarted}>Get Started to Have Fun</button>
      </div>
    </div>
  );
};

export default Splash;


//CSS
html, body {
    margin: 0;
    padding: 0;
    height: 100%;
    width: 100%;
  }
  
  .splash-container {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100vh; /* Full viewport height */
    width: 100vw; /* Full viewport width */
    background-color: #f5f5f5; /* Background color */
    text-align: center;
  }
  
  .splash-content {
    max-width: 500px; /* Limit the width of the content */
    padding: 20px;
    background: white;
    border-radius: 10px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }
  
  .splash-content h1 {
    font-size: 2.5rem;
    margin-bottom: 10px;
  }
  
  .splash-content p {
    font-size: 1.2rem;
    margin-bottom: 20px;
  }
  
  .get-started-button {
    padding: 10px 20px;
    font-size: 1rem;
    color: white;
    background-color: #007bff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }
  
  .get-started-button:hover {
    background-color: #0056b3;
  }



  //AuthCOntect

import React, { createContext, useContext, useState, useEffect } from 'react';
import axios from 'axios';

axios.defaults.baseURL = 'http://localhost:8080'; // Ensure this matches your backend server

const AuthContext = createContext();

export const useAuth = () => useContext(AuthContext);

export const AuthProvider = ({ children }) => {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        const loadUser = async () => {
            if (localStorage.token) {
                axios.defaults.headers.common['x-auth-token'] = localStorage.token;
            }

            try {
                const res = await axios.get('/api/auth/user');
                setUser(res.data);
            } catch (error) {
                console.error('Load User Error:', error.response ? error.response.data : error.message);
            }
            setLoading(false);
        };

        loadUser();
    }, []);

    const login = async (email, password) => {
        try {
            const res = await axios.post('/api/auth/login', { email, password });
            localStorage.setItem('token', res.data.token);
            axios.defaults.headers.common['x-auth-token'] = res.data.token;
            const userRes = await axios.get('/api/auth/user');
            setUser(userRes.data);
            return {status:true,message:"Login Success"};
        } catch (error) {
            console.error('Login Error:', error);
            return {status:false,message:error.response};
        }
    };

    const register = async (username, email, password) => {
        try {
            const res = await axios.post('/api/auth/register', { username, email, password });
            localStorage.setItem('token', res.data.token);
            axios.defaults.headers.common['x-auth-token'] = res.data.token;
            const userRes = await axios.get('/api/auth/user');
            setUser(userRes.data);
            return {status:true,message:"Register Success"};
        } catch (error) {
            console.error('Registration Error:', error.response ? error.response.data : error.message);
            return {status:false,message:error.response};
        }
    };

    const logout = () => {
        localStorage.removeItem('token');
        setUser(null);
        delete axios.defaults.headers.common['x-auth-token'];
    };

    return (
        <AuthContext.Provider value={{ user, login, register, logout, loading }}>
            {!loading && children}
        </AuthContext.Provider>
    );
};
