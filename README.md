# react-3
Custom hook'lar (maxsus hook'lar) React-da kodni qayta ishlatish, mantiqni ajratish va komponentlarni toza saqlash uchun eng yaxshi usuldir.

Quyida siz so'ragan barcha hook'lar hooks/ papkasi ichida alohida fayllarda, to'liq JSDoc hujjati va ilg'or amaliyotlar bilan taqdim etilgan. Oxirida esa ularning barchasini ko'rsatib beruvchi bitta yaxlit demo komponenti (DemoApp.js) joylashtirilgan.

1. hooks/useToggle.js
JavaScript
import { useState, useCallback } from 'react';

/**
 * Boolean (true/false) holatni osongina o'zgartirish uchun hook.
 * * @param {boolean} [initialValue=false] - Boshlang'ich qiymat
 * @returns {[boolean, function, function, function]} Holat va uni boshqaruvchi funksiyalar (toggle, setTrue, setFalse)
 */
export default function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => setValue((prev) => !prev), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return [value, toggle, setTrue, setFalse];
}
2. hooks/useLocalStorage.js
JavaScript
import { useState, useEffect } from 'react';

/**
 * Ma'lumotlarni localStorage'da avtomatik saqlash va o'qish uchun universal hook.
 * * @template T
 * @param {string} key - LocalStorage kalit nomi
 * @param {T} initialValue - Boshlang'ich qiymat (agar localStorage bo'sh bo'lsa)
 * @returns {[T, function]} State qiymati va uni yangilovchi funksiya
 */
export default function useLocalStorage(key, initialValue) {
  // Boshlang'ich qiymatni localStorage'dan tekshirib olish
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`LocalStorage [${key}] o'qishda xatolik:`, error);
      return initialValue;
    }
  });

  // State o'zgarganda localStorage'ga yozish
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    } catch (error) {
      console.error(`LocalStorage [${key}] yozishda xatolik:`, error);
    }
  }, [key, storedValue]);

  return [storedValue, setStoredValue];
}
3. hooks/useFetch.js
JavaScript
import { useState, useEffect } from 'react';

/**
 * Race condition va AbortController bilan jihozlangan universal API fetch hooki.
 * * @param {string} url - So'rov yuboriladigan API manzili
 * @returns {{ data: any, loading: boolean, error: string|null }} Yuklanish holati, xatolik va kelgan ma'lumot
 */
export default function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!url) return;

    const controller = new AbortController();
    const signal = controller.signal;

    setLoading(true);
    setError(null);

    fetch(url, { signal })
      .then((res) => {
        if (!res.ok) throw new Error(`Xatolik yuz berdi: ${res.status}`);
        return res.json();
      })
      .then((jsonData) => {
        setData(jsonData);
        setLoading(false);
      })
      .catch((err) => {
        if (err.name === 'AbortError') {
          console.log('Eski fetch so\'rovi bekor qilindi.');
        } else {
          setError(err.message);
          setLoading(false);
        }
      });

    // Cleanup: URL o'zgarganda eski so'rov abort qilinadi
    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}
4. hooks/useDebounce.js
JavaScript
import { useState, useEffect } from 'react';

/**
 * Tez o'zgaruvchan qiymatni belgilangan vaqtga kechiktirib beruvchi hook.
 * * @template T
 * @param {T} value - Asosiy tez o'zgaruvchi qiymat
 * @param {number} delay - Kechikish vaqti (millisekundlarda)
 * @returns {T} Kechiktirilgan (debounced) qiymat
 */
export default function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    // Agar foydalanuvchi yozishda davom etsa, eski taymer o'chadi
    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}
5. hooks/useOnClickOutside.js
JavaScript
import { useEffect } from 'react';

/**
 * Berilgan elementdan tashqariga bosilganda biror amalni bajaruvchi hook (Modal/Dropdown uchun).
 * * @param {React.RefObject} ref - Tashqarisi kuzatilayotgan element ref'i
 * @param {function} callback - Tashqariga bosilganda ishga tushuvchi funksiya
 */
export default function useOnClickOutside(ref, callback) {
  useEffect(() => {
    const listener = (event) => {
      // Agar bosilgan joy ref ichidagi element bo'lsa, hech narsa qilmaydi
      if (!ref.current || ref.current.contains(event.target)) {
        return;
      }
      callback(event);
    };

    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);

    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, callback]);
}
6. hooks/useMediaQuery.js
JavaScript
import { useState, useEffect } from 'react';

/**
 * CSS Media Query holatini JavaScript ichida kuzatish uchun responsive hook.
 * * @param {string} query - CSS media query sharti (masalan, '(max-width: 768px)')
 * @returns {boolean} Media query sharti mos kelayotganligi (true/false)
 */
export default function useMediaQuery(query) {
  const [matches, setMatches] = useState(() => window.matchMedia(query).matches);

  useEffect(() => {
    const media = window.matchMedia(query);
    
    if (media.matches !== matches) {
      setMatches(media.matches);
    }

    const listener = (e) => setMatches(e.matches);
    
    media.addEventListener('change', listener);
    return () => media.removeEventListener('change', listener);
  }, [query, matches]);

  return matches;
}
7. Demo Sahifa: DemoApp.js
Barcha yaratilgan hook'larni real misolda tekshirish va ishlatish uchun ushbu demo sahifadan foydalanishingiz mumkin:

JavaScript
import React, { useState, useRef } from 'react';
import useToggle from './hooks/useToggle';
import useLocalStorage from './hooks/useLocalStorage';
import useFetch from './hooks/useFetch';
import useDebounce from './hooks/useDebounce';
import useOnClickOutside from './hooks/useOnClickOutside';
import useMediaQuery from './hooks/useMediaQuery';
import './DemoApp.css';

export default function DemoApp() {
  // A. useMediaQuery
  const isMobile = useMediaQuery('(max-width: 768px)');

  // B. useToggle
  const [isOpen, toggleOpen, setOpenTrue, setOpenFalse] = useToggle(false);

  // C. useLocalStorage
  const [name, setName] = useLocalStorage('username', 'Jasur');

  // D. useDebounce va useFetch birgalikda (Foydalanuvchi ID bo'yicha qidiruv)
  const [searchId, setSearchId] = useState('1');
  const debouncedId = useDebounce(searchId, 500); // 500ms o'tgach fetch qiladi
  const { data, loading, error } = useFetch(
    debouncedId ? `https://jsonplaceholder.typicode.com/users/${debouncedId}` : null
  );

  // E. useOnClickOutside (Dropdown / Modal)
  const dropdownRef = useRef();
  const [dropdownVisible, toggleDropdown, , setDropdownFalse] = useToggle(false);
  useOnClickOutside(dropdownRef, setDropdownFalse);

  return (
    <div className="demo-container">
      <h1>Custom Hooks Demo Sahifasi</h1>
      
      {/* 1. useMediaQuery */}
      <div className="demo-card">
        <h3>1. useMediaQuery</h3>
        <p>Ekran holati: {isMobile ? '📱 Mobil rejim (<=768px)' : '💻 Desktop rejim (>768px)'}</p>
      </div>

      {/* 2. useToggle */}
      <div className="demo-card">
        <h3>2. useToggle</h3>
        <button onClick={toggleOpen}>Almashtirish (Toggle)</button>
        <button onClick={setOpenTrue}>True qilish</button>
        <button onClick={setOpenFalse}>False qilish</button>
        <p>Holat qiymati: <strong>{isOpen ? 'OCHIQ 🟢' : 'YOPIQ 🔴'}</strong></p>
      </div>

      {/* 3. useLocalStorage */}
      <div className="demo-card">
        <h3>3. useLocalStorage</h3>
        <input 
          type="text" 
          value={name} 
          onChange={(e) => setName(e.target.value)} 
          placeholder="Ism kiriting..."
        />
        <p>Sahifani yangilasangiz ham bu ism saqlanib qoladi: <strong>{name}</strong></p>
      </div>

      {/* 4. useOnClickOutside */}
      <div className="demo-card">
        <h3>4. useOnClickOutside</h3>
        <div ref={dropdownRef} className="dropdown-wrapper">
          <button onClick={toggleDropdown}>Dropdown Menyu</button>
          {dropdownVisible && (
            <div className="dropdown-box">
              <p>Menyu faol! Karta tashqarisiga yoki ekranning ixtiyoriy bo'sh joyiga bossangiz u yopiladi.</p>
            </div>
          )}
        </div>
      </div>

      {/* 5. useDebounce & useFetch */}
      <div className="demo-card">
        <h3>5. useDebounce & useFetch (Birgalikda)</h3>
        <label>Foydalanuvchi ID (1-10 gacha kiriting): </label>
        <input 
          type="number" 
          value={searchId} 
          onChange={(e) => setSearchId(e.target.value)}
          min="1" max="10"
        />
        <small style={{display:'block', color:'#666', marginTop: '4px'}}>
          Yozishdan to'xtaganingizdan so'ng 500ms kutiladi...
        </small>

        <div className="fetch-result-box">
          {loading && <p>⏳ API'dan ma'lumot yuklanmoqda...</p>}
          {error && <p style={{color: 'red'}}>❌ Xato: {error}</p>}
          {!loading && !error && data && (
            <div>
              <p><strong>Ism:</strong> {data.name}</p>
              <p><strong>Email:</strong> {data.email}</p>
              <p><strong>Shahar:</strong> {data.address?.city}</p>
            </div>
          )}
        </div>
      </div>
    </div>
  );
}
DemoApp.css (Tashqi stillar)
CSS
body {
  background-color: #f3f4f6;
  font-family: 'Segoe UI', system-ui, sans-serif;
  padding: 20px;
}
.demo-container {
  max-width: 650px;
  margin: 0 auto;
  background: #fff;
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.05);
}
h1 { text-align: center; color: #1f2937; margin-bottom: 25px; }
.demo-card {
  border: 1px solid #e5e7eb;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 15px;
  background: #f9fafb;
}
.demo-card h3 { margin-top: 0; color: #3b82f6; font-size: 1.1rem; }
button {
  padding: 8px 14px;
  margin-right: 6px;
  border: 1px solid #cbd5e1;
  background: white;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s;
}
button:hover { background: #f1f5f9; border-color: #94a3b8; }
input {
  padding: 8px 12px;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  font-size: 0.95rem;
  outline: none;
}
input:focus { border-color: #3b82f6; }
.dropdown-wrapper { position: relative; display: inline-block; }
.dropdown-box {
  position: absolute;
  top: 110%;
  left: 0;
  background: #1e293b;
  color: #fff;
  padding: 12px;
  border-radius: 6px;
  z-index: 10;
  width: 220px;
  font-size: 0.85rem;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}
.fetch-result-box {
  margin-top: 12px;
  padding: 12px;
  background: #fff;
  border-left: 4px solid #10b981;
  border-radius: 6px;
  box-shadow: inset 0 1px 3px rgba(0,0,0,0.02);
}
.fetch-result-box p { margin: 6px 0; font-size: 0.95rem; }
