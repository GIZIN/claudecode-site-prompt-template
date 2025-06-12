# プロジェクト初期構築テンプレート

このファイルはAI（Claude）が参照する構築手順書です。人間向けの使用方法はREADME.mdを参照してください。

### 設定ファイル

会社情報とGist URLは別ファイル（[CLAUDE_CONFIG.md](./CLAUDE_CONFIG.md)）で管理されています。
プロジェクト構築前に、CLAUDE_CONFIG.mdファイルに以下の情報が設定されていることを確認してください：

1. **会社情報**
   - 会社名（日本語・英語）
   - 会社ドメイン
   - メールアドレス設定

2. **Gist URLs**
   - 会社概要ページ
   - プライバシーポリシー
   - 利用規約

### 設定の確認方法

1. `CLAUDE_CONFIG.md`を開く
2. 会社情報セクションで自社の情報が設定されているか確認
3. Gist URLsセクションで各ページのURLが設定されているか確認
4. 必要に応じて値を更新

※ 英語版URLが提供されない場合（「なし」や空欄の場合）、日本語版から自動翻訳して英語ページを生成します。

### 置換が必要な箇所

プロジェクト構築時は、`CLAUDE_CONFIG.md`の置換ルールに従って以下の箇所の値を置き換えます：

1. **辞書ファイル** (`dictionaries/ja.json`, `dictionaries/en.json`) - 会社名の表示
2. **環境変数のデフォルト値** (`.env.local.example`) - 会社情報、メール設定、Gist URL
3. **README.mdのライセンス表記** - 会社名（英語）

---

## 構築前の確認事項

構築を開始する前に、以下の情報を教えてください：

### 前提条件

このテンプレートを使用するには以下のツールが必要です：

1. **Node.js** (v18以上)

### 必要な情報

構築を開始する前に、以下の情報を教えてください：

1. **プロジェクト名**（英数字、ハイフン使用可）
   - 例: `service-landing`, `corporate-site`
   - 注意: このプロジェクト名がフォルダ名になります

2. **サービス名**（日本語・英語）
   - 日本語: 高速開発サービス
   - 英語: Fast Development Service

3. **多言語対応**
   - 日英の多言語対応が必要ですか？（はい/いいえ）
   - 「いいえ」の場合は日本語のみのサイトを構築します

4. **メールアドレス設定**
   - 送信元メールアドレス: `CLAUDE_CONFIG.md`の設定を使用（デフォルト: noreply@gizin.co.jp）
   - お問い合わせ受信用メールアドレス: `CLAUDE_CONFIG.md`の設定を使用（デフォルト: contact@gizin.co.jp）

---

### プロジェクトの作成方法

```bash
npx create-next-app@latest [プロジェクト名] --typescript --tailwind --app --no-src-dir --eslint --import-alias "@/*"
cd [プロジェクト名]
```

**重要な注意**: 
- 上記のコマンドで`[プロジェクト名]`という名前のフォルダが作成されます
- `--eslint`フラグでESLintを有効化（推奨）
- `--import-alias "@/*"`でインポートエイリアスを設定
- `cd [プロジェクト名]`でそのフォルダに移動してから、以降の手順を実行します
- 以降の手順で作成するファイルは、すべてこのプロジェクトフォルダ内に作成します

**インタラクティブな質問を回避するための注意**:
- 最新のcreate-next-appでは、上記のフラグを指定してもTurbopackについて質問される場合があります
- その場合は以下のコマンドを使用してください：
```bash
echo "n" | npx create-next-app@latest [プロジェクト名] --typescript --tailwind --app --no-src-dir --eslint --import-alias "@/*"
```

### Tailwind CSS v3への移行（重要）

最新のcreate-next-appはTailwind CSS v4をインストールしますが、互換性の問題があるため、v3に変更する必要があります：

```bash
cd [プロジェクト名]
npm uninstall @tailwindcss/postcss tailwindcss
npm install -D tailwindcss@^3.4.0 postcss autoprefixer
```

また、以下のTailwind CSS設定ファイルを作成します：

`tailwind.config.js`:
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

`postcss.config.js`:
```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

## 構築指示

### 固定値（プロジェクト共通設定）

会社情報とGist URLは [CLAUDE_CONFIG.md](./CLAUDE_CONFIG.md) で管理されています。

対応言語: 日本語（ja）、英語（en）

重要: プロジェクト構築前に、CLAUDE_CONFIG.mdファイルの内容を確認し、必要に応じて更新してください。

上記の情報を確認した後、以下の手順で構築を行います：

### 1. 多言語対応の基本設定

#### i18n設定ファイル
`i18n-config.ts`:
```typescript
export const i18n = {
  defaultLocale: 'ja',
  locales: ['ja', 'en'],
} as const;

export type Locale = (typeof i18n)['locales'][number];
```

#### ミドルウェア設定
`middleware.ts`:
```typescript
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { i18n } from './i18n-config';

export function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname;
  const pathnameIsMissingLocale = i18n.locales.every(
    (locale) => !pathname.startsWith(`/${locale}/`) && pathname !== `/${locale}`
  );

  if (pathnameIsMissingLocale) {
    const locale = i18n.defaultLocale;
    return NextResponse.redirect(
      new URL(`/${locale}${pathname}`, request.url)
    );
  }
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

### 2. ディレクトリ構造

以下は、create-next-appで作成したプロジェクトのルートディレクトリ内に作成する構造です：

```
./ (プロジェクトのルートディレクトリ)
├── app/
│   ├── [lang]/
│   │   ├── layout.tsx      # 言語別レイアウト
│   │   ├── page.tsx        # トップページ
│   │   ├── company/
│   │   │   └── page.tsx    # 会社概要
│   │   ├── privacy/
│   │   │   └── page.tsx    # プライバシーポリシー
│   │   ├── terms/
│   │   │   └── page.tsx    # 利用規約
│   │   ├── contact/
│   │   │   └── page.tsx    # お問い合わせ
│   │   ├── loading.tsx     # ローディング画面
│   │   ├── error.tsx       # エラーページ
│   │   └── not-found.tsx   # 404ページ
│   ├── api/
│   │   └── contact/
│   │       └── route.ts    # お問い合わせAPI
│   ├── globals.css
│   ├── layout.tsx          # ルートレイアウト（最小限）
│   └── page.tsx            # リダイレクト用
├── components/
│   ├── Header.tsx          # 言語切替含む
│   ├── Footer.tsx
│   ├── LanguageSwitcher.tsx
│   ├── MobileMenu.tsx     # モバイルメニュー
│   └── ContactForm.tsx    # お問い合わせフォーム
├── dictionaries/
│   ├── ja.json            # 日本語辞書
│   └── en.json            # 英語辞書
├── lib/
│   ├── getDictionary.ts   # 辞書取得関数
│   ├── resend.ts          # Resend設定
│   ├── validations.ts     # バリデーション
│   └── rateLimit.ts       # レート制限
├── types/
│   └── dictionary.ts      # 辞書の型定義
├── public/
├── i18n-config.ts
├── middleware.ts
├── .env.local.example
├── .editorconfig
├── package.json
└── README.md
```

### 3. 辞書ファイル

**重要な仕様:**
- フッターのコピーライト表記は日英両方で英語に統一されています
- 日本語ページでも `© [会社名（英語）]. All rights reserved.` と表示されます

`dictionaries/ja.json`:
```json
{
  "nav": {
    "home": "ホーム",
    "company": "会社概要",
    "privacy": "プライバシーポリシー",
    "terms": "利用規約",
    "contact": "お問い合わせ"
  },
  "company": {
    "name": "[会社名（日本語）]",
    "nameEn": "[会社名（英語）]"
  },
  "footer": {
    "copyright": "© [会社名（英語）]. All rights reserved."
  },
  "contact": {
    "title": "お問い合わせ",
    "description": "お気軽にお問い合わせください",
    "form": {
      "name": "お名前",
      "email": "メールアドレス",
      "company": "会社名",
      "phone": "電話番号",
      "subject": "件名",
      "message": "お問い合わせ内容",
      "submit": "送信する",
      "submitting": "送信中...",
      "required": "必須"
    },
    "success": "お問い合わせを受け付けました。担当者より連絡させていただきます。",
    "error": "送信に失敗しました。もう一度お試しください。"
  }
}
```

`dictionaries/en.json`:
```json
{
  "nav": {
    "home": "Home",
    "company": "Company",
    "privacy": "Privacy Policy",
    "terms": "Terms of Service",
    "contact": "Contact"
  },
  "company": {
    "name": "[会社名（英語）]",
    "nameEn": "[会社名（英語）]"
  },
  "footer": {
    "copyright": "© [会社名（英語）]. All rights reserved."
  },
  "contact": {
    "title": "Contact Us",
    "description": "Feel free to contact us",
    "form": {
      "name": "Name",
      "email": "Email",
      "company": "Company",
      "phone": "Phone",
      "subject": "Subject",
      "message": "Message",
      "submit": "Submit",
      "submitting": "Submitting...",
      "required": "Required"
    },
    "success": "Your inquiry has been received. We will contact you soon.",
    "error": "Failed to send. Please try again."
  }
}
```

### 4. 辞書取得関数

`lib/getDictionary.ts`:
```typescript
import 'server-only';
import type { Locale } from '@/i18n-config';

const dictionaries = {
  ja: () => import('@/dictionaries/ja.json').then((module) => module.default),
  en: () => import('@/dictionaries/en.json').then((module) => module.default),
};

export const getDictionary = async (locale: Locale) => dictionaries[locale]();
```

### 5. バリデーション設定（多言語対応）

`lib/validations.ts`:
```typescript
import { z } from 'zod';

export const createContactSchema = (lang: 'ja' | 'en') => {
  const messages = {
    ja: {
      nameRequired: '名前は必須です',
      emailInvalid: '有効なメールアドレスを入力してください',
      subjectRequired: '件名は必須です',
      messageMin: 'お問い合わせ内容は10文字以上で入力してください',
    },
    en: {
      nameRequired: 'Name is required',
      emailInvalid: 'Please enter a valid email address',
      subjectRequired: 'Subject is required',
      messageMin: 'Message must be at least 10 characters',
    },
  };

  const msg = messages[lang];

  return z.object({
    name: z.string().min(1, msg.nameRequired),
    email: z.string().email(msg.emailInvalid),
    company: z.string().optional(),
    phone: z.string().optional(),
    subject: z.string().min(1, msg.subjectRequired),
    message: z.string().min(10, msg.messageMin),
  });
};

export const contactSchema = createContactSchema('ja'); // デフォルト（API用）

export type ContactFormData = z.infer<typeof contactSchema>;
```

※ バリデーションメッセージも言語に応じて切り替わるように実装しています。

### 6. Resend設定

`lib/resend.ts`:
```typescript
import { Resend } from 'resend';

export const resend = new Resend(process.env.RESEND_API_KEY);
```

### 7. レート制限の実装

`lib/rateLimit.ts`:
```typescript
import { NextRequest } from 'next/server';

const rateLimitMap = new Map();

export function rateLimit(options: {
  uniqueTokenPerInterval?: number;
  interval?: number;
} = {}) {
  const {
    uniqueTokenPerInterval = 500,
    interval = 60000,
  } = options;

  return {
    check: (req: NextRequest, limit: number, token: string) =>
      new Promise<void>((resolve, reject) => {
        const tokenCount = rateLimitMap.get(token) || [0, Date.now()];
        tokenCount[0] += 1;

        const currentUsage = tokenCount[0];
        const resetTime = tokenCount[1];

        if (resetTime + interval <= Date.now()) {
          tokenCount[0] = 1;
          tokenCount[1] = Date.now();
          rateLimitMap.set(token, tokenCount);
          resolve();
        } else if (currentUsage <= limit) {
          rateLimitMap.set(token, tokenCount);
          resolve();
        } else {
          reject(new Error('Rate limit exceeded'));
        }
      }),
  };
}
```

### 8. お問い合わせAPI

`app/api/contact/route.ts`:
```typescript
import { NextResponse } from 'next/server';
import { NextRequest } from 'next/server';
import { resend } from '@/lib/resend';
import { contactSchema } from '@/lib/validations';
import { rateLimit } from '@/lib/rateLimit';

const limiter = rateLimit({
  interval: 60 * 1000, // 1分
  uniqueTokenPerInterval: 500,
});

export async function POST(request: Request) {
  try {
    // レート制限チェック
    const forwarded = request.headers.get('x-forwarded-for');
    const ip = forwarded ? forwarded.split(',')[0] : 'anonymous';

    try {
      await limiter.check(request as NextRequest, 5, ip); // 1分間に5リクエストまで
    } catch {
      return NextResponse.json(
        { error: 'Too many requests' },
        { status: 429 }
      );
    }

    const body = await request.json();
    const validatedData = contactSchema.parse(body);

    const { data, error } = await resend.emails.send({
      from: process.env.RESEND_FROM_EMAIL!,
      to: process.env.CONTACT_EMAIL!,
      subject: `【お問い合わせ】${validatedData.subject}`,
      html: `
        <h2>お問い合わせを受け付けました</h2>
        <p><strong>お名前:</strong> ${validatedData.name}</p>
        <p><strong>メールアドレス:</strong> ${validatedData.email}</p>
        ${validatedData.company ? `<p><strong>会社名:</strong> ${validatedData.company}</p>` : ''}
        ${validatedData.phone ? `<p><strong>電話番号:</strong> ${validatedData.phone}</p>` : ''}
        <p><strong>件名:</strong> ${validatedData.subject}</p>
        <p><strong>お問い合わせ内容:</strong></p>
        <p>${validatedData.message.replace(/\n/g, '<br>')}</p>
      `,
    });

    if (error) {
      return NextResponse.json({ error: 'Failed to send email' }, { status: 500 });
    }

    return NextResponse.json({ success: true });
  } catch (error) {
    return NextResponse.json({ error: 'Invalid request' }, { status: 400 });
  }
}
```

### 9. お問い合わせフォームコンポーネント

`components/ContactForm.tsx`:
```typescript
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import type { ContactFormData } from '@/lib/validations';
import { createContactSchema } from '@/lib/validations';

interface ContactFormProps {
  dict: any;
  lang: string;
}

export default function ContactForm({ dict, lang }: ContactFormProps) {
  const router = useRouter();
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [formData, setFormData] = useState<ContactFormData>({
    name: '',
    email: '',
    company: '',
    phone: '',
    subject: '',
    message: '',
  });

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);
    setErrors({});

    // クライアントサイドバリデーション
    const schema = createContactSchema(lang as 'ja' | 'en');
    const result = schema.safeParse(formData);

    if (!result.success) {
      const fieldErrors: Record<string, string> = {};
      result.error.issues.forEach((issue) => {
        if (issue.path[0]) {
          fieldErrors[issue.path[0] as string] = issue.message;
        }
      });
      setErrors(fieldErrors);
      setIsSubmitting(false);
      return;
    }

    try {
      const response = await fetch('/api/contact', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      });

      if (response.ok) {
        alert(dict.contact.success);
        setFormData({
          name: '',
          email: '',
          company: '',
          phone: '',
          subject: '',
          message: '',
        });
      } else {
        alert(dict.contact.error);
      }
    } catch (error) {
      alert(dict.contact.error);
    } finally {
      setIsSubmitting(false);
    }
  };

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));
  };

  return (
    <form onSubmit={handleSubmit} className="max-w-2xl mx-auto space-y-6" noValidate>
      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div>
          <label htmlFor="name" className="block text-sm font-medium mb-2">
            {dict.contact.form.name} <span className="text-red-500">*</span>
          </label>
          <input
            type="text"
            id="name"
            name="name"
            value={formData.name}
            onChange={handleChange}
            className={`w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 ${errors.name ? 'border-red-500' : ''}`}
          />
          {errors.name && (
            <p className="mt-1 text-sm text-red-600">{errors.name}</p>
          )}
        </div>

        <div>
          <label htmlFor="email" className="block text-sm font-medium mb-2">
            {dict.contact.form.email} <span className="text-red-500">*</span>
          </label>
          <input
            type="email"
            id="email"
            name="email"
            value={formData.email}
            onChange={handleChange}
            className={`w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 ${errors.email ? 'border-red-500' : ''}`}
          />
          {errors.email && (
            <p className="mt-1 text-sm text-red-600">{errors.email}</p>
          )}
        </div>

        <div>
          <label htmlFor="company" className="block text-sm font-medium mb-2">
            {dict.contact.form.company}
          </label>
          <input
            type="text"
            id="company"
            name="company"
            value={formData.company}
            onChange={handleChange}
            className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
          />
        </div>

        <div>
          <label htmlFor="phone" className="block text-sm font-medium mb-2">
            {dict.contact.form.phone}
          </label>
          <input
            type="tel"
            id="phone"
            name="phone"
            value={formData.phone}
            onChange={handleChange}
            className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
          />
        </div>
      </div>

      <div>
        <label htmlFor="subject" className="block text-sm font-medium mb-2">
          {dict.contact.form.subject} <span className="text-red-500">*</span>
        </label>
        <input
          type="text"
          id="subject"
          name="subject"
          value={formData.subject}
          onChange={handleChange}
          className={`w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 ${errors.subject ? 'border-red-500' : ''}`}
        />
        {errors.subject && (
          <p className="mt-1 text-sm text-red-600">{errors.subject}</p>
        )}
      </div>

      <div>
        <label htmlFor="message" className="block text-sm font-medium mb-2">
          {dict.contact.form.message} <span className="text-red-500">*</span>
        </label>
        <textarea
          id="message"
          name="message"
          value={formData.message}
          onChange={handleChange}
          rows={6}
          className={`w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 ${errors.message ? 'border-red-500' : ''}`}
        />
        {errors.message && (
          <p className="mt-1 text-sm text-red-600">{errors.message}</p>
        )}
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full py-3 px-6 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50 transition-colors"
      >
        {isSubmitting ? dict.contact.form.submitting : dict.contact.form.submit}
      </button>
    </form>
  );
}
```

### 10. お問い合わせページ

`app/[lang]/contact/page.tsx`:
```typescript
import { getDictionary } from '@/lib/getDictionary';
import ContactForm from '@/components/ContactForm';
import type { Locale } from '@/i18n-config';

export async function generateMetadata({
  params: { lang }
}: {
  params: { lang: Locale }
}) {
  const dict = await getDictionary(lang);

  return {
    title: dict.contact.title,
    description: dict.contact.description,
  };
}

export default async function ContactPage({
  params: { lang }
}: {
  params: { lang: Locale }
}) {
  const dict = await getDictionary(lang);

  return (
    <div className="container mx-auto px-4 sm:px-6 lg:px-8 py-16">
      <h1 className="text-3xl font-bold text-center mb-4">
        {dict.contact.title}
      </h1>
      <p className="text-center text-gray-600 dark:text-gray-400 mb-12">
        {dict.contact.description}
      </p>

      <ContactForm dict={dict} lang={lang} />
    </div>
  );
}
```

### 11. 必須ページの自動生成

各言語に対応したページを作成：

**会社概要ページ**
- 作成先: app/[lang]/company/page.tsx
- 日本語版Gist: CLAUDE_CONFIG.mdの「会社概要」日本語URLを使用
- 英語版Gist: CLAUDE_CONFIG.mdの「会社概要」英語URLを使用

`app/[lang]/company/page.tsx`:
```typescript
import { getDictionary } from '@/lib/getDictionary';
import type { Locale } from '@/i18n-config';
import { marked } from 'marked';

export async function generateMetadata({
  params: { lang }
}: {
  params: { lang: Locale }
}) {
  const dict = await getDictionary(lang);
  return {
    title: dict.nav.company,
    description: dict.nav.company,
  };
}

export default async function CompanyPage({
  params: { lang }
}: {
  params: { lang: Locale }
}) {
  const dict = await getDictionary(lang);
  let content = '';

  try {
    // Gist URLからコンテンツを取得
    const gistUrl = lang === 'ja'
      ? process.env.COMPANY_GIST_JA_URL
      : process.env.COMPANY_GIST_EN_URL;

    if (gistUrl && gistUrl !== 'なし') {
      const response = await fetch(gistUrl, {
        next: { revalidate: 3600 } // 1時間キャッシュ
      });

      if (response.ok) {
        const text = await response.text();
        content = await marked(text);
        // 最初のh1タグを削除（重複を防ぐため）
        content = content.replace(/<h1[^>]*>.*?<\/h1>/, '');
      }
    } else if (lang === 'en' && process.env.COMPANY_GIST_JA_URL) {
      // 英語版がない場合は日本語版から自動翻訳するロジックを追加
      // ※ 実装時には翻訳APIや事前に翻訳したコンテンツを使用
      content = '<p>Company information will be available soon.</p>';
    }
  } catch (error) {
    console.error('Failed to fetch company content:', error);
  }

  return (
    <div className="container mx-auto px-4 sm:px-6 lg:px-8 py-16">
      <h1 className="text-3xl font-bold mb-8">{dict.nav.company}</h1>
      {content ? (
        <div className="prose max-w-none" dangerouslySetInnerHTML={{ __html: content }} />
      ) : (
        <p className="text-gray-600 dark:text-gray-400">
          {lang === 'ja' ? 'コンテンツを読み込んでいます...' : 'Loading content...'}
        </p>
      )}
    </div>
  );
}
```

**プライバシーポリシー**
- 作成先: app/[lang]/privacy/page.tsx
- 日本語版Gist: CLAUDE_CONFIG.mdの「プライバシーポリシー」日本語URLを使用
- 英語版Gist: CLAUDE_CONFIG.mdの「プライバシーポリシー」英語URLを使用

`app/[lang]/privacy/page.tsx`:
```typescript
import { getDictionary } from '@/lib/getDictionary';
import type { Locale } from '@/i18n-config';
import { marked } from 'marked';

export async function generateMetadata({
  params: { lang }
}: {
  params: { lang: Locale }
}) {
  const dict = await getDictionary(lang);
  return {
    title: dict.nav.privacy,
    description: dict.nav.privacy,
  };
}

export default async function PrivacyPage({
  params: { lang }
}: {
  params: { lang: Locale }
}) {
  const dict = await getDictionary(lang);
  let content = '';

  try {
    const gistUrl = lang === 'ja'
      ? process.env.PRIVACY_GIST_JA_URL
      : process.env.PRIVACY_GIST_EN_URL;

    if (gistUrl && gistUrl !== 'なし') {
      const response = await fetch(gistUrl, {
        next: { revalidate: 3600 }
      });

      if (response.ok) {
        const text = await response.text();
        content = await marked(text);
        // 最初のh1タグを削除（重複を防ぐため）
        content = content.replace(/<h1[^>]*>.*?<\/h1>/, '');
      }
    } else if (lang === 'en' && process.env.PRIVACY_GIST_JA_URL) {
      content = '<p>Privacy policy will be available soon.</p>';
    }
  } catch (error) {
    console.error('Failed to fetch privacy content:', error);
  }

  return (
    <div className="container mx-auto px-4 sm:px-6 lg:px-8 py-16">
      <h1 className="text-3xl font-bold mb-8">{dict.nav.privacy}</h1>
      {content ? (
        <div className="prose max-w-none" dangerouslySetInnerHTML={{ __html: content }} />
      ) : (
        <p className="text-gray-600 dark:text-gray-400">
          {lang === 'ja' ? 'コンテンツを読み込んでいます...' : 'Loading content...'}
        </p>
      )}
    </div>
  );
}
```

**利用規約**
- 作成先: app/[lang]/terms/page.tsx
- 日本語版Gist: CLAUDE_CONFIG.mdの「利用規約」日本語URLを使用
- 英語版Gist: CLAUDE_CONFIG.mdの「利用規約」英語URLを使用

`app/[lang]/terms/page.tsx`:
```typescript
import { getDictionary } from '@/lib/getDictionary';
import type { Locale } from '@/i18n-config';
import { marked } from 'marked';

export async function generateMetadata({
  params: { lang }
}: {
  params: { lang: Locale }
}) {
  const dict = await getDictionary(lang);
  return {
    title: dict.nav.terms,
    description: dict.nav.terms,
  };
}

export default async function TermsPage({
  params: { lang }
}: {
  params: { lang: Locale }
}) {
  const dict = await getDictionary(lang);
  let content = '';

  try {
    const gistUrl = lang === 'ja'
      ? process.env.TERMS_GIST_JA_URL
      : process.env.TERMS_GIST_EN_URL;

    if (gistUrl && gistUrl !== 'なし') {
      const response = await fetch(gistUrl, {
        next: { revalidate: 3600 }
      });

      if (response.ok) {
        const text = await response.text();
        content = await marked(text);
        // 最初のh1タグを削除（重複を防ぐため）
        content = content.replace(/<h1[^>]*>.*?<\/h1>/, '');
      }
    } else if (lang === 'en' && process.env.TERMS_GIST_JA_URL) {
      content = '<p>Terms of service will be available soon.</p>';
    }
  } catch (error) {
    console.error('Failed to fetch terms content:', error);
  }

  return (
    <div className="container mx-auto px-4 sm:px-6 lg:px-8 py-16">
      <h1 className="text-3xl font-bold mb-8">{dict.nav.terms}</h1>
      {content ? (
        <div className="prose max-w-none" dangerouslySetInnerHTML={{ __html: content }} />
      ) : (
        <p className="text-gray-600 dark:text-gray-400">
          {lang === 'ja' ? 'コンテンツを読み込んでいます...' : 'Loading content...'}
        </p>
      )}
    </div>
  );
}
```

※ 各ページは言語パラメータ（lang）に基づいて、対応するGistの内容を表示します。

**Gistコンテンツの重要な仕様:**
- Gistから取得したMarkdownコンテンツの**最初のh1タグは自動的に削除されます**
- これはページ側で統一的にタイトルを表示するため、タイトルの重複を防ぐための仕様です
- Gistには本文のみを記載し、h1タイトルが含まれていても問題ありません（自動削除されます）

**英語版ページが提供されていない場合の対応:**
- 日本語版のGist URLのみが提供された場合、自動的に日本語版の内容を英語に翻訳してページを生成します
- 翻訳は法的文書として適切な品質を保ちながら、自然な英語表現を使用します
- 会社名、住所、メールアドレスなどの固有名詞は翻訳せずそのまま使用します

### 12. モバイルメニューコンポーネント（言語切替を含む）

`components/MobileMenu.tsx`:
```typescript
'use client';

import { useState } from 'react';
import Link from 'next/link';
import { usePathname } from 'next/navigation';
import type { Locale } from '@/i18n-config';
import { i18n } from '@/i18n-config';

interface MobileMenuProps {
  dict: any;
  lang: Locale;
}

export default function MobileMenu({ dict, lang }: MobileMenuProps) {
  const [isOpen, setIsOpen] = useState(false);
  const pathName = usePathname();

  const redirectedPathName = (locale: string) => {
    if (!pathName) return '/';
    const segments = pathName.split('/');
    segments[1] = locale;
    return segments.join('/');
  };

  return (
    <div>
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="p-2 text-gray-700 dark:text-gray-300"
        aria-label="Menu"
      >
        <svg className="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path
            strokeLinecap="round"
            strokeLinejoin="round"
            strokeWidth={2}
            d={isOpen ? 'M6 18L18 6M6 6l12 12' : 'M4 6h16M4 12h16M4 18h16'}
          />
        </svg>
      </button>

      {isOpen && (
        <div className="absolute top-16 left-0 right-0 bg-white dark:bg-gray-900 border-b border-gray-200 dark:border-gray-700 shadow-lg">
          <nav className="flex flex-col space-y-4 p-4">
            <Link
              href={`/${lang}`}
              className="text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400"
              onClick={() => setIsOpen(false)}
              scroll={true}
            >
              {dict.nav.home}
            </Link>
            <Link
              href={`/${lang}/company`}
              className="text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400"
              onClick={() => setIsOpen(false)}
              scroll={true}
            >
              {dict.nav.company}
            </Link>
            <Link
              href={`/${lang}/contact`}
              className="text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400"
              onClick={() => setIsOpen(false)}
              scroll={true}
            >
              {dict.nav.contact}
            </Link>

            <div className="pt-4 border-t border-gray-200 dark:border-gray-700">
              <div className="flex gap-2">
                {i18n.locales.map((locale) => (
                  <Link
                    key={locale}
                    href={redirectedPathName(locale)}
                    className={`px-3 py-1 rounded ${
                      lang === locale
                        ? 'bg-blue-600 text-white'
                        : 'bg-gray-100 dark:bg-gray-800 text-gray-700 dark:text-gray-300'
                    }`}
                    onClick={() => setIsOpen(false)}
                  >
                    {locale === 'ja' ? '日本語' : 'English'}
                  </Link>
                ))}
              </div>
            </div>
          </nav>
        </div>
      )}
    </div>
  );
}
```

※ 言語切替機能をハンバーガーメニューの中に統合しました。選択中の言語は青色でハイライトされます。

### 13. 言語切替コンポーネント

`components/LanguageSwitcher.tsx`:
```typescript
'use client';

import { usePathname } from 'next/navigation';
import Link from 'next/link';
import { i18n } from '@/i18n-config';

export default function LanguageSwitcher() {
  const pathName = usePathname();
  const redirectedPathName = (locale: string) => {
    if (!pathName) return '/';
    const segments = pathName.split('/');
    segments[1] = locale;
    return segments.join('/');
  };

  return (
    <div className="flex gap-2">
      {i18n.locales.map((locale) => (
        <Link
          key={locale}
          href={redirectedPathName(locale)}
          className="px-3 py-1 rounded text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-800"
        >
          {locale === 'ja' ? '日本語' : 'English'}
        </Link>
      ))}
    </div>
  );
}
```

### 13. 環境変数

`.env.local.example`:
```env
# サイト情報
NEXT_PUBLIC_SITE_NAME_JA=[サービス名（日本語）]
NEXT_PUBLIC_SITE_NAME_EN=[サービス名（英語）]
NEXT_PUBLIC_SITE_URL=https://[会社ドメイン]

# 会社情報（CLAUDE_CONFIG.mdの値で置換）
NEXT_PUBLIC_COMPANY_NAME_JA=[会社名（日本語）]
NEXT_PUBLIC_COMPANY_NAME_EN=[会社名（英語）]
NEXT_PUBLIC_COMPANY_DOMAIN=[会社ドメイン]

# Resend設定
RESEND_API_KEY=re_xxxxxxxxxxxxx
RESEND_FROM_EMAIL=[送信元メールアドレス]
CONTACT_EMAIL=[お問い合わせ用メールアドレス]

# Gist URL設定（CLAUDE_CONFIG.mdの値で置換）
COMPANY_GIST_JA_URL=[会社概要日本語URL]
COMPANY_GIST_EN_URL=[会社概要英語URL]
PRIVACY_GIST_JA_URL=[プライバシーポリシー日本語URL]
PRIVACY_GIST_EN_URL=[プライバシーポリシー英語URL]
TERMS_GIST_JA_URL=[利用規約日本語URL]
TERMS_GIST_EN_URL=[利用規約英語URL]
```

### 14. グローバルスタイル

`app/globals.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* 横スクロール防止 */
* {
  max-width: 100vw;
}

html, body {
  overflow-x: hidden;
  width: 100%;
}

/* 言語切替時のスムーズな遷移 */
html {
  scroll-behavior: smooth;
}

/* スティッキーヘッダーのためのスペース確保 */
body {
  padding-top: 64px; /* ヘッダーの高さ分 */
}

/* ページ内リンクのオフセット調整 */
:target {
  scroll-margin-top: 80px;
}

/* フォーカススタイル */
@layer utilities {
  .focus-visible-ring {
    @apply focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:ring-offset-2;
  }
}

/* Markdown用のスタイル */
.prose {
  @apply max-w-none;
}

.prose h1 {
  @apply text-3xl font-bold mt-8 mb-4;
}

.prose h2 {
  @apply text-2xl font-bold mt-6 mb-3;
}

.prose h3 {
  @apply text-xl font-bold mt-4 mb-2;
}

.prose p {
  @apply mb-4 leading-relaxed;
}

.prose ul {
  @apply list-disc list-inside mb-4 space-y-2;
}

.prose ol {
  @apply list-decimal list-inside mb-4 space-y-2;
}

.prose a {
  @apply text-blue-600 hover:text-blue-700 underline;
}

.prose blockquote {
  @apply border-l-4 border-gray-300 pl-4 italic my-4;
}

.prose code {
  @apply bg-gray-100 dark:bg-gray-800 px-1 py-0.5 rounded text-sm;
}

.prose pre {
  @apply bg-gray-100 dark:bg-gray-800 p-4 rounded-lg overflow-x-auto mb-4;
}

.prose pre code {
  @apply bg-transparent p-0;
}

.prose table {
  @apply w-full border-collapse mb-4;
}

.prose th {
  @apply border border-gray-300 px-4 py-2 bg-gray-50 dark:bg-gray-800 font-semibold;
}

.prose td {
  @apply border border-gray-300 px-4 py-2;
}
```

### 15. ヘッダーコンポーネント

`components/Header.tsx`:
```typescript
import Link from 'next/link';
import LanguageSwitcher from './LanguageSwitcher';
import MobileMenu from './MobileMenu';
import { getDictionary } from '@/lib/getDictionary';
import type { Locale } from '@/i18n-config';

export default async function Header({ lang }: { lang: Locale }) {
  const dict = await getDictionary(lang);

  return (
    <header className="fixed top-0 left-0 right-0 z-50 bg-white dark:bg-gray-900 border-b border-gray-200 dark:border-gray-700">
      <div className="container mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex justify-between items-center h-16">
          <Link href={`/${lang}`} className="font-bold text-xl text-gray-900 dark:text-white" scroll={true}>
            {lang === 'ja'
              ? process.env.NEXT_PUBLIC_SITE_NAME_JA
              : process.env.NEXT_PUBLIC_SITE_NAME_EN}
          </Link>

          <nav className="hidden md:flex space-x-8">
            <Link href={`/${lang}`} className="text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400" scroll={true}>
              {dict.nav.home}
            </Link>
            <Link href={`/${lang}/company`} className="text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400" scroll={true}>
              {dict.nav.company}
            </Link>
            <Link href={`/${lang}/contact`} className="text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400" scroll={true}>
              {dict.nav.contact}
            </Link>
          </nav>

          <div className="flex items-center gap-4">
            <div className="hidden md:block">
              <LanguageSwitcher />
            </div>
            <div className="md:hidden">
              <MobileMenu dict={dict} lang={lang} />
            </div>
          </div>
        </div>
      </div>
    </header>
  );
}
```

### 16. フッターコンポーネント

`components/Footer.tsx`:
```typescript
import Link from 'next/link';
import { getDictionary } from '@/lib/getDictionary';
import type { Locale } from '@/i18n-config';

export default async function Footer({ lang }: { lang: Locale }) {
  const dict = await getDictionary(lang);

  return (
    <footer className="bg-gray-100 dark:bg-gray-800 mt-auto">
      <div className="container mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
          <div>
            <h3 className="font-bold mb-4 text-gray-900 dark:text-white">{dict.company.name}</h3>
            <p className="text-sm text-gray-600 dark:text-gray-400">
              {lang === 'ja'
                ? '革新的なソリューションを提供します'
                : 'Providing innovative solutions'}
            </p>
          </div>

          <div>
            <h4 className="font-semibold mb-4 text-gray-900 dark:text-white">
              {lang === 'ja' ? 'リンク' : 'Links'}
            </h4>
            <ul className="space-y-2 text-sm">
              <li>
                <Link href={`/${lang}/company`} className="text-gray-600 dark:text-gray-400 hover:text-blue-600 dark:hover:text-blue-400" scroll={true}>
                  {dict.nav.company}
                </Link>
              </li>
              <li>
                <Link href={`/${lang}/contact`} className="text-gray-600 dark:text-gray-400 hover:text-blue-600 dark:hover:text-blue-400" scroll={true}>
                  {dict.nav.contact}
                </Link>
              </li>
              <li>
                <Link href={`/${lang}/privacy`} className="text-gray-600 dark:text-gray-400 hover:text-blue-600 dark:hover:text-blue-400" scroll={true}>
                  {dict.nav.privacy}
                </Link>
              </li>
              <li>
                <Link href={`/${lang}/terms`} className="text-gray-600 dark:text-gray-400 hover:text-blue-600 dark:hover:text-blue-400" scroll={true}>
                  {dict.nav.terms}
                </Link>
              </li>
            </ul>
          </div>
        </div>

        <div className="mt-8 pt-8 border-t border-gray-300 dark:border-gray-700 text-center text-sm text-gray-600 dark:text-gray-400">
          {dict.footer.copyright}
        </div>
      </div>
    </footer>
  );
}
```

### 17. ルートレイアウト

`app/[lang]/layout.tsx`:
```typescript
import { Inter } from 'next/font/google';
import { i18n, type Locale } from '@/i18n-config';
import Header from '@/components/Header';
import Footer from '@/components/Footer';
import { getDictionary } from '@/lib/getDictionary';
import '../globals.css';

const inter = Inter({ subsets: ['latin'] });

export async function generateStaticParams() {
  return i18n.locales.map((locale) => ({ lang: locale }));
}

export async function generateMetadata({
  params: { lang }
}: {
  params: { lang: Locale }
}) {
  const isJa = lang === 'ja';

  return {
    title: {
      template: `%s | ${isJa ? process.env.NEXT_PUBLIC_COMPANY_NAME_JA : process.env.NEXT_PUBLIC_COMPANY_NAME_EN}`,
      default: isJa
        ? `${process.env.NEXT_PUBLIC_SITE_NAME_JA} | ${process.env.NEXT_PUBLIC_COMPANY_NAME_JA}`
        : `${process.env.NEXT_PUBLIC_SITE_NAME_EN} | ${process.env.NEXT_PUBLIC_COMPANY_NAME_EN}`,
    },
    description: isJa
      ? `${process.env.NEXT_PUBLIC_SITE_NAME_JA}の公式サイト`
      : `Official website of ${process.env.NEXT_PUBLIC_SITE_NAME_EN}`,
    openGraph: {
      locale: lang === 'ja' ? 'ja_JP' : 'en_US',
      alternateLocale: lang === 'ja' ? 'en_US' : 'ja_JP',
    },
  };
}

export default function RootLayout({
  children,
  params,
}: {
  children: React.ReactNode;
  params: { lang: Locale };
}) {
  return (
    <html lang={params.lang}>
      <body className={`${inter.className} min-h-screen flex flex-col`}>
        <Header lang={params.lang} />
        <main className="flex-grow">{children}</main>
        <Footer lang={params.lang} />
      </body>
    </html>
  );
}
```

### 18. トップページのテンプレート

`app/[lang]/page.tsx`:
```typescript
import { getDictionary } from '@/lib/getDictionary';
import type { Locale } from '@/i18n-config';

export default async function HomePage({
  params: { lang }
}: {
  params: { lang: Locale }
}) {
  const dict = await getDictionary(lang);
  const isJa = lang === 'ja';

  return (
    <div className="container mx-auto px-4 sm:px-6 lg:px-8 py-16">
      <h1 className="text-4xl font-bold text-center mb-8">
        {isJa
          ? `${process.env.NEXT_PUBLIC_SITE_NAME_JA}へようこそ`
          : `Welcome to ${process.env.NEXT_PUBLIC_SITE_NAME_EN}`}
      </h1>

      <p className="text-center text-gray-600 dark:text-gray-400">
        {isJa
          ? 'ここにサービスの説明を記載します'
          : 'Service description goes here'}
      </p>
    </div>
  );
}
```

### 19. リダイレクト設定

`app/page.tsx`:
```typescript
import { redirect } from 'next/navigation';
import { i18n } from '@/i18n-config';

export default function RootPage() {
  redirect(`/${i18n.defaultLocale}`);
}
```

`app/layout.tsx`:
```typescript
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return children;
}
```

### 20. 404ページ

`app/[lang]/not-found.tsx`:
```typescript
import Link from 'next/link';
import { getDictionary } from '@/lib/getDictionary';
import type { Locale } from '@/i18n-config';

export default async function NotFound({
  params: { lang }
}: {
  params: { lang: Locale }
}) {
  const dict = await getDictionary(lang);

  return (
    <div className="container mx-auto px-4 py-16 text-center">
      <h1 className="text-4xl font-bold mb-4">404</h1>
      <p className="text-gray-600 dark:text-gray-400 mb-8">
        {lang === 'ja' ? 'ページが見つかりません' : 'Page not found'}
      </p>
      <Link
        href={`/${lang}`}
        className="text-blue-600 hover:text-blue-700"
      >
        {lang === 'ja' ? 'ホームに戻る' : 'Back to home'}
      </Link>
    </div>
  );
}
```

### 21. ローディングページ

`app/[lang]/loading.tsx`:
```typescript
export default function Loading() {
  return (
    <div className="flex items-center justify-center min-h-[60vh]">
      <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
    </div>
  );
}
```

### 22. エラーページ

`app/[lang]/error.tsx`:
```typescript
'use client';

import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    console.error(error);
  }, [error]);

  return (
    <div className="container mx-auto px-4 py-16 text-center">
      <h2 className="text-2xl font-bold mb-4">エラーが発生しました</h2>
      <button
        onClick={() => reset()}
        className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        もう一度試す
      </button>
    </div>
  );
}
```

### 23. package.json依存関係

```json
{
  "name": "[プロジェクト名]",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "15.x",
    "react": "^19",
    "react-dom": "^19",
    "resend": "^4.0.0",
    "zod": "^3.22.0",
    "server-only": "^0.0.1",
    "marked": "^15.0.0"
  },
  "devDependencies": {
    "@types/node": "^20",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "typescript": "^5",
    "tailwindcss": "^3.4.0",
    "postcss": "^8",
    "autoprefixer": "^10",
    "concurrently": "^8.2.0",
    "wait-on": "^7.0.0",
    "open-cli": "^8.0.0"
  }
}
```

### 24. README.md

```markdown
# [プロジェクト名]

[サービス名（日本語）] / [サービス名（英語）]

## 概要

[会社名（日本語）]が提供する[サービスの説明]

## 技術スタック

- Next.js 14 (App Router)
- TypeScript
- Tailwind CSS v3
- Resend (メール送信)
- Zod (バリデーション)

## セットアップ

### 1. 依存関係のインストール

\`\`\`bash
npm install

# 注意: Tailwind CSS v4がインストールされている場合は、v3に変更
npm uninstall @tailwindcss/postcss tailwindcss
npm install -D tailwindcss@^3.4.0 postcss autoprefixer
\`\`\`

### 2. 環境変数の設定

\`\`\`bash
cp .env.local.example .env.local
\`\`\`

以下の環境変数を設定してください：

- `NEXT_PUBLIC_SITE_NAME_JA`: サービス名（日本語）
- `NEXT_PUBLIC_SITE_NAME_EN`: サービス名（英語）
- `RESEND_API_KEY`: ResendのAPIキー
- `CONTACT_EMAIL`: お問い合わせ受信用メールアドレス

### 3. 開発サーバーの起動

\`\`\`bash
npm run dev
\`\`\`

http://localhost:3000 でアプリケーションが起動します。

## ディレクトリ構造

\`\`\`
├── app/              # Next.js App Router
├── components/       # Reactコンポーネント
├── dictionaries/     # 多言語対応辞書
├── lib/             # ユーティリティ関数
└── public/          # 静的ファイル
\`\`\`

## 多言語対応

- 日本語: `/ja/*`
- 英語: `/en/*`

## デプロイ

### Vercel

\`\`\`bash
vercel
\`\`\`

環境変数を本番環境用に設定してください。

## License

© [会社名（英語）]. All rights reserved.
\`\`\`

### 25. エディタ設定

`.editorconfig`:
```ini
# EditorConfig is awesome: https://EditorConfig.org

# top-most EditorConfig file
root = true

# Unix-style newlines with a newline ending every file
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
indent_style = space
indent_size = 2
trim_trailing_whitespace = true

# Markdown files
[*.md]
trim_trailing_whitespace = false

# Package files
[{package.json,*.yml}]
indent_size = 2

# TypeScript/JavaScript files
[*.{ts,tsx,js,jsx}]
indent_size = 2
```

### 26. PostCSS設定

`postcss.config.js`:
```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

### 27. TypeScript設定

`tsconfig.json`:
```json
{
  "compilerOptions": {
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 28. Next.js設定

`next.config.js`:
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {}

module.exports = nextConfig
```

### 29. Tailwind CSS設定

`tailwind.config.js`:
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

### 30. ESLint設定

`.eslintrc.json`:
```json
{
  "extends": ["next/core-web-vitals"],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn"
  }
}
```

### 31. Prettier設定

`.prettierrc`:
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2
}
```

### 32. 型定義ファイル

`types/dictionary.ts`:
```typescript
export interface Dictionary {
  nav: {
    home: string;
    company: string;
    privacy: string;
    terms: string;
    contact: string;
  };
  company: {
    name: string;
    nameEn: string;
  };
  footer: {
    copyright: string;
  };
  contact: {
    title: string;
    description: string;
    form: {
      name: string;
      email: string;
      company: string;
      phone: string;
      subject: string;
      message: string;
      submit: string;
      submitting: string;
      required: string;
    };
    success: string;
    error: string;
  };
}
```

### 33. アクセシビリティ改善版のコンタクトフォーム

`components/ContactForm.tsx`の改善版（アクセシビリティ対応）:
```typescript
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import type { ContactFormData } from '@/lib/validations';

interface ContactFormProps {
  dict: any;
  lang: string;
}

export default function ContactForm({ dict, lang }: ContactFormProps) {
  const router = useRouter();
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [errors, setErrors] = useState<Partial<ContactFormData>>({});
  const [formData, setFormData] = useState<ContactFormData>({
    name: '',
    email: '',
    company: '',
    phone: '',
    subject: '',
    message: '',
  });

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);
    setErrors({});

    try {
      const response = await fetch('/api/contact', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      });

      if (response.ok) {
        alert(dict.contact.success);
        setFormData({
          name: '',
          email: '',
          company: '',
          phone: '',
          subject: '',
          message: '',
        });
      } else {
        alert(dict.contact.error);
      }
    } catch (error) {
      alert(dict.contact.error);
    } finally {
      setIsSubmitting(false);
    }
  };

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));
  };

  return (
    <form onSubmit={handleSubmit} className="max-w-2xl mx-auto space-y-6" noValidate>
      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div>
          <label htmlFor="name" className="block text-sm font-medium mb-2">
            {dict.contact.form.name} <span className="text-red-500" aria-label={dict.contact.form.required}>*</span>
          </label>
          <input
            type="text"
            id="name"
            name="name"
            value={formData.name}
            onChange={handleChange}
            required
            aria-required="true"
            aria-describedby="name-error"
            className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
          />
          {errors.name && (
            <p id="name-error" className="mt-1 text-sm text-red-600" role="alert">
              {errors.name}
            </p>
          )}
        </div>

        <div>
          <label htmlFor="email" className="block text-sm font-medium mb-2">
            {dict.contact.form.email} <span className="text-red-500" aria-label={dict.contact.form.required}>*</span>
          </label>
          <input
            type="email"
            id="email"
            name="email"
            value={formData.email}
            onChange={handleChange}
            required
            aria-required="true"
            aria-describedby="email-error"
            className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
          />
          {errors.email && (
            <p id="email-error" className="mt-1 text-sm text-red-600" role="alert">
              {errors.email}
            </p>
          )}
        </div>

        <div>
          <label htmlFor="company" className="block text-sm font-medium mb-2">
            {dict.contact.form.company}
          </label>
          <input
            type="text"
            id="company"
            name="company"
            value={formData.company}
            onChange={handleChange}
            className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
          />
        </div>

        <div>
          <label htmlFor="phone" className="block text-sm font-medium mb-2">
            {dict.contact.form.phone}
          </label>
          <input
            type="tel"
            id="phone"
            name="phone"
            value={formData.phone}
            onChange={handleChange}
            className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
          />
        </div>
      </div>

      <div>
        <label htmlFor="subject" className="block text-sm font-medium mb-2">
          {dict.contact.form.subject} <span className="text-red-500" aria-label={dict.contact.form.required}>*</span>
        </label>
        <input
          type="text"
          id="subject"
          name="subject"
          value={formData.subject}
          onChange={handleChange}
          required
          aria-required="true"
          className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
        />
      </div>

      <div>
        <label htmlFor="message" className="block text-sm font-medium mb-2">
          {dict.contact.form.message} <span className="text-red-500" aria-label={dict.contact.form.required}>*</span>
        </label>
        <textarea
          id="message"
          name="message"
          value={formData.message}
          onChange={handleChange}
          required
          aria-required="true"
          rows={6}
          className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none"
        />
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        aria-busy={isSubmitting}
        className="w-full py-3 px-6 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50 transition-colors focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
      >
        {isSubmitting ? dict.contact.form.submitting : dict.contact.form.submit}
      </button>
    </form>
  );
}
```

### 34. パフォーマンス最適化

`next.config.js`の改善版:
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    domains: ['gist.githubusercontent.com'],
    formats: ['image/avif', 'image/webp'],
  },
  experimental: {
    optimizeFonts: true,
  },
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production',
  },
}

module.exports = nextConfig
```

### 35. SEO強化版のメタデータ

`app/[lang]/layout.tsx`の改善版:
```typescript
export async function generateMetadata({
  params: { lang }
}: {
  params: { lang: Locale }
}) {
  const isJa = lang === 'ja';

  return {
    title: {
      template: `%s | ${isJa ? process.env.NEXT_PUBLIC_COMPANY_NAME_JA : process.env.NEXT_PUBLIC_COMPANY_NAME_EN}`,
      default: isJa
        ? `${process.env.NEXT_PUBLIC_SITE_NAME_JA} | ${process.env.NEXT_PUBLIC_COMPANY_NAME_JA}`
        : `${process.env.NEXT_PUBLIC_SITE_NAME_EN} | ${process.env.NEXT_PUBLIC_COMPANY_NAME_EN}`,
    },
    description: isJa
      ? `${process.env.NEXT_PUBLIC_SITE_NAME_JA}の公式サイト`
      : `Official website of ${process.env.NEXT_PUBLIC_SITE_NAME_EN}`,
    keywords: isJa
      ? ['日本語', 'キーワード', 'SEO']
      : ['English', 'Keywords', 'SEO'],
    authors: [{ name: process.env.NEXT_PUBLIC_COMPANY_NAME_JA }],
    creator: process.env.NEXT_PUBLIC_COMPANY_NAME_JA,
    publisher: process.env.NEXT_PUBLIC_COMPANY_NAME_JA,
    formatDetection: {
      email: false,
      address: false,
      telephone: false,
    },
    metadataBase: new URL(process.env.NEXT_PUBLIC_SITE_URL || 'https://example.com'),
    alternates: {
      canonical: '/',
      languages: {
        'ja': '/ja',
        'en': '/en',
      },
    },
    openGraph: {
      title: isJa
        ? `${process.env.NEXT_PUBLIC_SITE_NAME_JA} | ${process.env.NEXT_PUBLIC_COMPANY_NAME_JA}`
        : `${process.env.NEXT_PUBLIC_SITE_NAME_EN} | ${process.env.NEXT_PUBLIC_COMPANY_NAME_EN}`,
      description: isJa
        ? `${process.env.NEXT_PUBLIC_SITE_NAME_JA}の公式サイト`
        : `Official website of ${process.env.NEXT_PUBLIC_SITE_NAME_EN}`,
      url: process.env.NEXT_PUBLIC_SITE_URL,
      siteName: isJa
        ? process.env.NEXT_PUBLIC_SITE_NAME_JA
        : process.env.NEXT_PUBLIC_SITE_NAME_EN,
      locale: lang === 'ja' ? 'ja_JP' : 'en_US',
      alternateLocale: lang === 'ja' ? 'en_US' : 'ja_JP',
      type: 'website',
    },
    twitter: {
      card: 'summary_large_image',
      title: isJa
        ? `${process.env.NEXT_PUBLIC_SITE_NAME_JA} | ${process.env.NEXT_PUBLIC_COMPANY_NAME_JA}`
        : `${process.env.NEXT_PUBLIC_SITE_NAME_EN} | ${process.env.NEXT_PUBLIC_COMPANY_NAME_EN}`,
      description: isJa
        ? `${process.env.NEXT_PUBLIC_SITE_NAME_JA}の公式サイト`
        : `Official website of ${process.env.NEXT_PUBLIC_SITE_NAME_EN}`,
    },
    robots: {
      index: true,
      follow: true,
      googleBot: {
        index: true,
        follow: true,
        'max-video-preview': -1,
        'max-image-preview': 'large',
        'max-snippet': -1,
      },
    },
  };
}

`.prettierrc`:
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2
}
```

---

## 構築フロー

1. **初期質問フェーズ**: 
   - プロジェクト名、サービス名（日英）、多言語対応、メールアドレスを確認
2. **構築フェーズ**: 多言語対応の基本構造とお問い合わせフォームを生成
3. **開発サーバー起動案内フェーズ**: 開発サーバーの起動方法を案内
4. **完了フェーズ**: 動作確認事項と次のステップを案内

## 構築完了後の処理

プロジェクト生成後、以下のコマンドを自動実行：

```bash
# 依存関係のインストール
npm install

# .env.local ファイルの作成
cp .env.local.example .env.local

# 開発サーバーの起動案内
echo ""
echo "🚀 セットアップが完了しました！"
echo ""
echo "開発サーバーを起動するには："
echo "npm run dev"
echo ""
echo "サーバーが起動したら、ブラウザで以下のURLにアクセスしてください："
echo "http://localhost:3000"
echo ""
echo "※ ポート3000が使用中の場合は、自動的に別のポート（3001, 3002...）が使用されます"
```

### 開発サーバーの起動について

Next.js の開発サーバーは以下の特徴があります：

1. **ポート自動検出**: デフォルトでポート3000を使用しますが、使用中の場合は自動的に別のポート（3001, 3002...）を使用します
2. **起動メッセージ**: サーバーが起動すると、使用しているポート番号とURLがコンソールに表示されます
3. **ブラウザ起動**: 手動でブラウザを開いて表示されたURLにアクセスしてください

例：
```
▲ Next.js 15.x.x
- Local:        http://localhost:3000
- Network:      http://192.168.x.x:3000

✓ Starting...
✓ Ready in 2.1s
```


## 構築後の確認事項

### 動作確認
- 開発サーバーが表示するURL（例: `http://localhost:3000`）にアクセス
- 自動的に `/ja` にリダイレクトされるか確認
- `/en` で英語版が表示されるか確認
- 言語切替が正常に動作するか確認
- お問い合わせフォームが正常に動作するか確認

### 必須ページの確認
- `/ja/company` と `/en/company`
- `/ja/privacy` と `/en/privacy`
- `/ja/terms` と `/en/terms`
- `/ja/contact` と `/en/contact`

### Resendの設定確認
- Resendアカウントの作成
- APIキーの取得と `.env.local` への設定
- 送信元ドメインの認証（DNSレコード設定）

### 次のステップの案内
```
基本的な多言語対応サイトの構築が完了しました！

実装済みの機能：
✅ 多言語対応（日本語・英語）
✅ お問い合わせフォーム（Resend使用）
✅ 会社概要・プライバシーポリシー・利用規約ページ

開発サーバーを起動するには：
1. プロジェクトディレクトリで npm run dev を実行
2. 表示されたURL（通常は http://localhost:3000）にアクセス

次に実装できる機能：
- SEO最適化（hreflangタグなど）
- 特定のページコンテンツの実装
- お問い合わせフォームの拡張（ファイル添付など）
- デプロイ設定（Vercel、Netlifyなど）

何を実装しますか？
```

## 技術スタック

- Next.js 15 (App Router)
- TypeScript
- Tailwind CSS v3（注意: v4ではなくv3を使用）
- 多言語対応（i18n）
- Resend（メール送信）
- Zod（バリデーション）

## 注意事項

### Next.js 15 対応
- 動的ルートの`params`は非同期で取得する必要があります
- `params: Promise<{ lang: Locale }>`の形式で受け取り、`await params`で展開
- not-found.tsxではparamsが提供されないため、別の方法で言語を取得

### Tailwind CSS
- **必ずv3を使用**（v4はNext.js 15.3.3と互換性の問題があります）
- `postcss.config.js`では`tailwindcss: {}`を使用（`@tailwindcss/postcss`ではない）

### 多言語対応
- すべてのテキストは辞書ファイルから取得する
- 新しいページを追加する際は必ず両言語分を作成
- URLは常に言語プレフィックス付きで構成
- 画像のaltテキストも多言語対応する

### Resend使用時の注意
1. **ドメイン認証**
   - Resendで送信元ドメインの認証が必要
   - DNSレコード（SPF、DKIM）の設定が必要

2. **開発環境**
   - 開発時はResendのテストモードを使用
   - `.env.local`にAPIキーを設定

3. **本番環境**
   - 本番用のAPIキーを使用
   - レート制限に注意（無料プランは1日100通まで）

### 開発サーバーの起動

Next.jsの開発サーバーは自動的にポートを検出して起動します：

```bash
npm run dev
```

- デフォルトポート: 3000
- ポートが使用中の場合: 自動的に3001, 3002...と空いているポートを探します
- 起動後、コンソールに実際のURLが表示されます

### 注意事項

#### Tailwind CSS のバージョン
- **必ずTailwind CSS v3を使用してください**（v4はNext.js 15との互換性の問題があります）
- create-next-appでv4がインストールされた場合は、以下のコマンドでv3に変更：
  ```bash
  npm uninstall @tailwindcss/postcss tailwindcss
  npm install -D tailwindcss@^3.4.0 postcss autoprefixer
  ```

#### Next.js 15での動的ルート
- 動的ルートの`params`は非同期で取得する必要があります：
  ```typescript
  export default async function Page({
    params
  }: {
    params: Promise<{ lang: Locale }>
  }) {
    const { lang } = await params;
    // ...
  }
  ```

#### セキュリティ考慮事項
- 環境変数は必ず`.env.local`に記載し、`.gitignore`に含める
- お問い合わせフォームにはレート制限を実装済み
- 本番環境では追加のCSRF対策を検討

## よくある問題と解決方法

### 1. デスクトップでの言語切替表示

**問題**: 言語切替がモバイルのハンバーガーメニューにのみ表示される

**解決方法**: ヘッダーコンポーネントを以下のように修正：

```typescript
<div className="flex items-center gap-4">
  <div className="hidden md:block">
    <LanguageSwitcher />
  </div>
  <div className="md:hidden">
    <MobileMenu dict={dict} lang={lang} />
  </div>
</div>
```

### 2. バリデーションメッセージの多言語対応

**問題**: 英語ページでも日本語のバリデーションメッセージが表示される

**解決方法**:

1. クライアントサイドで言語に応じたスキーマを使用：
```typescript
const schema = createContactSchema(lang as 'ja' | 'en');
const result = schema.safeParse(formData);
```

2. HTML5バリデーションを無効化：
```typescript
<form onSubmit={handleSubmit} className="max-w-2xl mx-auto space-y-6" noValidate>
```

3. カスタムエラーメッセージの表示：
```typescript
{errors.name && (
  <p className="mt-1 text-sm text-red-600">{errors.name}</p>
)}
```

