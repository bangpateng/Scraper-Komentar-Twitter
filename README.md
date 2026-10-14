# Scraper-Komentar-Twitter

1. Open Link Retweet
2. F12 Console
3. Paste

```
// =====================================================================
//  Scraper komentar/balasan postingan X (Twitter)
//  Cara pakai: buka postingan di browser (sudah login), tekan F12 →
//  tab Console → paste seluruh script ini → Enter. Biarkan sampai selesai.
// =====================================================================
(async () => {
  const MAX_IDLE_ROUNDS = 8;     // berhenti setelah 8x scroll tanpa komentar baru
  const DELAY_MS = 1800;         // jeda antar scroll (naikkan jika internet lambat)

  const sleep = (ms) => new Promise((r) => setTimeout(r, ms));
  const mainId = (location.pathname.match(/status\/(\d+)/) || [])[1];
  const results = new Map();

  // Tombol "Show more replies" / "Show probable spam" / versi Bahasa Indonesia
  const MORE_BTN = /show (more )?repl|show probable spam|show additional|tampilkan (lebih banyak |)balasan|tampilkan balasan lainnya|kemungkinan spam/i;

  function clickMoreButtons() {
    document.querySelectorAll('div[role="button"], button, span').forEach((el) => {
      const t = (el.innerText || '').trim();
      if (t.length < 60 && MORE_BTN.test(t)) {
        try { (el.closest('[role="button"], button') || el).click(); } catch (e) {}
      }
    });
  }

  function collect() {
    let added = 0;
    document.querySelectorAll('article[data-testid="tweet"]').forEach((art) => {
      const timeLink = art.querySelector('a[href*="/status/"] time')?.closest('a');
      if (!timeLink) return;
      const m = timeLink.getAttribute('href').match(/^\/([^/]+)\/status\/(\d+)/);
      if (!m) return;
      const [, handle, id] = m;
      if (id === mainId || results.has(id)) return;

      const textEl = art.querySelector('div[data-testid="tweetText"]');
      const text = textEl ? textEl.innerText.replace(/\s+/g, ' ').trim() : '';
      const name = art.querySelector('div[data-testid="User-Name"] span')?.innerText || '';
      const wallets = (text.match(/0x[a-fA-F0-9]{40}/g) || []).join(' ');
      const hasMedia = !!art.querySelector('[data-testid="tweetPhoto"], video');
      const time = timeLink.querySelector('time')?.getAttribute('datetime') || '';

      results.set(id, {
        username: '@' + handle,
        name,
        text,
        wallet: wallets,
        media: hasMedia ? 'ya' : '',
        waktu: time,
        link: 'https://x.com/' + handle + '/status/' + id,
      });
      added++;
    });
    return added;
  }

  console.log('%c⏳ Mulai scraping… jangan tutup tab ini.', 'color:orange;font-weight:bold');
  let idle = 0;
  while (idle < MAX_IDLE_ROUNDS) {
    clickMoreButtons();
    const added = collect();
    idle = added ? 0 : idle + 1;
    console.log(`Terkumpul: ${results.size} komentar (+${added})`);
    window.scrollBy(0, window.innerHeight * 1.5);
    await sleep(DELAY_MS);
  }

  const rows = [...results.values()];

  // 1) Format teks siap paste ke Claude
  const plain = rows.map((r) =>
    `${r.username}: ${r.text}${r.media ? ' [ada gambar/video]' : ''}`
  ).join('\n');

  // 2) CSV untuk Excel / Google Sheets
  const esc = (v) => `"${String(v).replace(/"/g, '""')}"`;
  const headers = Object.keys(rows[0] || { username: '' });
  const csv = '\uFEFF' + [headers.join(','), ...rows.map((r) => headers.map((h) => esc(r[h])).join(','))].join('\n');

  const download = (content, filename, type) => {
    const a = document.createElement('a');
    a.href = URL.createObjectURL(new Blob([content], { type }));
    a.download = filename;
    document.body.appendChild(a);
    a.click();
    a.remove();
  };
  download(plain, `komentar_${mainId}.txt`, 'text/plain;charset=utf-8');
  download(csv, `komentar_${mainId}.csv`, 'text/csv;charset=utf-8');

  try { copy(plain); console.log('📋 Teks juga sudah disalin ke clipboard.'); } catch (e) {}

  console.table(rows.map(({ username, text }) => ({ username, text: text.slice(0, 80) })));
  console.log(`%c✅ Selesai! ${rows.length} komentar tersimpan (file .txt dan .csv terunduh).`,
    'color:limegreen;font-weight:bold');
})();
```
4. Tunggu Sampai Selesai, Copy Username
