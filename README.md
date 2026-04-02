# Scraper-Komentar-Twitter

1. Open Link Retweet
2. F12 Console
3. Paste

```
window.__xUsers = window.__xUsers || new Set();

(async () => {
  const delay = ms => new Promise(r => setTimeout(r, ms));
  let scrollAttempts = 0;
  const MAX_SCROLL = 150;
  const MIN_KATA = 6;

  const collect = () => {
    document.querySelectorAll('article[data-testid="tweet"]').forEach(article => {
      const tweetText = article.querySelector('[data-testid="tweetText"]');
      if (!tweetText) return;
      const kataAsli = tweetText.innerText.trim().split(/\s+/)
        .filter(w => !w.startsWith('@') && !w.startsWith('http') && w.length > 0);
      if (kataAsli.length < MIN_KATA) return;
      const userLink = article.querySelector('[data-testid="User-Name"] a[href^="/"]');
      const m = userLink?.getAttribute('href')?.match(/^\/([A-Za-z0-9_]{1,15})$/);
      if (m) window.__xUsers.add('@' + m[1]);
    });
  };

  console.log('%c🚀 X Scraper v4 mulai...', 'color:#1d9bf0;font-weight:bold;font-size:14px');
  console.log('%cFilter: komentar minimal 6 kata (mention & link tidak dihitung)', 'color:gray;font-size:12px');

  while (scrollAttempts < MAX_SCROLL) {
    const prev = window.__xUsers.size;
    collect();
    window.scrollBy({ top: 1500, behavior: 'smooth' });
    await delay(2000);
    collect();
    window.__xUsers.size === prev ? scrollAttempts++ : scrollAttempts = 0;
    if (scrollAttempts % 10 === 0)
      console.log(`📊 Lolos filter: ${window.__xUsers.size} | Scroll kosong: ${scrollAttempts}/${MAX_SCROLL}`);
  }

  const hasil = [...window.__xUsers];

  console.log('%c✅ SELESAI!', 'color:green;font-size:18px;font-weight:bold');
  console.log('%cTotal username lolos filter: ' + hasil.length, 'color:green;font-size:14px');
  console.log('%c--- DAFTAR USERNAME (select semua → klik kanan → Copy) ---', 'color:orange;font-weight:bold');
  console.log(hasil.join('\n'));
  console.log('%c--- SELESAI ---', 'color:orange;font-weight:bold');
})();
```

4. Tunggu Sampai Selesai, Copy Username
