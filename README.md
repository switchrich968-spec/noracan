# 'use client'

import posthog from 'posthog-js'
import { PostHogProvider } from 'posthog-js/react'
import { ReactNode } from 'react'

if (typeof window !== 'undefined' && process.env.NEXT_PUBLIC_POSTHOG_KEY) {
  posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY, {
    api_host: process.env.NEXT_PUBLIC_POSTHOG_HOST || 'https://us.i.posthog.com',
    person_profiles: 'identified_only',
    loaded: (posthog) => {
      if (process.env.NODE_ENV === 'development') posthog.debug()
    },
  })
}

export function Providers({ children }: { children: ReactNode }) {
  return <PostHogProvider client={posthog}>{children}</PostHogProvider>
}

import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import './globals.css'
import { Providers } from './providers'
import CookieConsent from 'react-cookie-consent'

const inter = Inter({ subsets: ['latin'] })

export const metadata: Metadata = {
  title: 'Noracan — AI Automation Platform',
  description: 'Build business automations by chatting with AI. Edit visually. Deploy in minutes.',
  keywords: ['automation', 'AI', 'workflows', 'no-code', 'business automation'],
}

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className="dark">
      <body className={`${inter.className} bg-black text-white antialiased`}>
        <Providers>{children}</Providers>
        <CookieConsent
          location="bottom"
          style={{ background: '#18181b', borderTop: '1px solid rgba(255,255,255,0.1)' }}
          buttonStyle={{
            background: '#a855f7',
            color: 'white',
            fontSize: '14px',
            fontWeight: '600',
            padding: '10px 20px',
            borderRadius: '8px',
          }}
          expires={150}
        >
          We use cookies to improve your experience. By continuing, you agree to our{' '}
          <a href="/privacy" style={{ color: '#a855f7', textDecoration: 'underline' }}>
            Privacy Policy
          </a>
          .
        </CookieConsent>
      </body>
    </html>
  )
}

import { Metadata } from 'next'
import Link from 'next/link'
import { Sparkles, ArrowRight, CheckCircle2 } from 'lucide-react'

export const metadata: Metadata = {
  title: 'Join the Noracan Beta',
  description: 'Be among the first to build automations with AI.',
}

export default function BetaPage() {
  return (
    <div className="min-h-screen bg-black text-white flex flex-col">
      <nav className="border-b border-white/10 px-6 py-4">
        <Link href="/" className="flex items-center gap-2 font-bold text-xl">
          <Sparkles className="h-6 w-6 text-purple-400" />
          <span>Noracan</span>
        </Link>
      </nav>

      <div className="flex-1 flex items-center justify-center px-6 py-20">
        <div className="max-w-2xl text-center">
          <div className="inline-flex items-center gap-2 rounded-full border border-white/10 bg-white/5 px-4 py-1.5 text-sm text-gray-300 mb-8">
            <span className="h-2 w-2 rounded-full bg-green-400 animate-pulse" />
            <span>Beta now open — limited spots</span>
          </div>

          <h1 className="text-5xl md:text-6xl font-bold tracking-tight mb-6">
            Build automations by{' '}
            <span className="bg-gradient-to-r from-purple-400 to-pink-400 bg-clip-text text-transparent">
              chatting with AI
            </span>
          </h1>

          <p className="text-xl text-gray-400 mb-10 max-w-xl mx-auto">
            Describe what you want in plain English. Noracan builds it. You stay in control.
          </p>

          <form action="/api/beta-signup" method="POST" className="max-w-md mx-auto space-y-4">
            <input
              type="email"
              name="email"
              placeholder="your@email.com"
              required
              className="w-full px-5 py-4 rounded-xl bg-white/5 border border-white/10 text-white placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-purple-500/50 focus:border-purple-500 transition"
            />
            <button
              type="submit"
              className="w-full flex items-center justify-center gap-2 px-5 py-4 rounded-xl bg-purple-600 hover:bg-purple-700 text-white font-semibold text-lg transition shadow-lg shadow-purple-500/20"
            >
              Request Beta Access
              <ArrowRight className="h-5 w-5" />
            </button>
          </form>

          <div className="mt-12 grid grid-cols-1 md:grid-cols-3 gap-6 text-left">
            {[
              { title: 'AI-first', desc: 'Describe workflows in plain English' },
              { title: 'Visual editor', desc: 'Fine-tune with drag-and-drop' },
              { title: '10+ integrations', desc: 'Gmail, Slack, Notion, and more' },
            ].map((item) => (
              <div key={item.title} className="p-4 rounded-xl border border-white/10 bg-white/[0.02]">
                <CheckCircle2 className="h-5 w-5 text-purple-400 mb-2" />
                <h3 className="font-semibold text-white mb-1">{item.title}</h3>
                <p className="text-sm text-gray-400">{item.desc}</p>
              </div>
            ))}
          </div>

          <p className="mt-10 text-sm text-gray-500">
            Free during beta. No credit card required.
          </p>
        </div>
      </div>
    </div>
  )
}

import { NextRequest, NextResponse } from 'next/server'
import { createClient } from '@/lib/supabase/server'

export async function POST(request: NextRequest) {
  try {
    const formData = await request.formData()
    const email = formData.get('email') as string

    if (!email || !email.includes('@')) {
      return NextResponse.redirect(new URL('/beta?error=invalid_email', request.url))
    }

    const supabase = createClient()

    const { error } = await supabase.from('beta_signups').insert({
      email,
      source: 'beta_page',
      created_at: new Date().toISOString(),
    })

    if (error) {
      console.error('Beta signup error:', error)
    }

    return NextResponse.redirect(new URL('/beta/thanks', request.url))
  } catch (error) {
    console.error('Beta signup error:', error)
    return NextResponse.redirect(new URL('/beta?error=unknown', request.url))
  }
}

import Link from 'next/link'
import { CheckCircle2 } from 'lucide-react'

export default function BetaThanksPage() {
  return (
    <div className="min-h-screen bg-black text-white flex items-center justify-center px-6">
      <div className="max-w-md text-center">
        <div className="inline-flex p-4 rounded-full bg-green-500/10 border border-green-500/20 mb-6">
          <CheckCircle2 className="h-10 w-10 text-green-400" />
        </div>
        <h1 className="text-4xl font-bold mb-4">You're on the list!</h1>
        <p className="text-gray-400 mb-8">
          We'll email you within 48 hours with your beta access. Check your inbox (and spam folder).
        </p>
        <div className="space-y-3 text-left p-6 rounded-xl border border-white/10 bg-white/[0.02]">
          <h2 className="font-semibold text-white mb-3">While you wait:</h2>
          <ul className="space-y-2 text-sm text-gray-400">
            <li>• Think about 3 workflows you'd automate first</li>
            <li>• List the apps you use daily (Gmail, Slack, etc.)</li>
            <li>• Reply to our welcome email with questions</li>
          </ul>
        </div>
        <Link
          href="/"
          className="mt-8 inline-block text-purple-400 hover:text-purple-300 text-sm transition"
        >
          ← Back to homepage
        </Link>
      </div>
    </div>
  )
}

import Link from 'next/link'
import { Sparkles } from 'lucide-react'

export default function TermsPage() {
  return (
    <div className="min-h-screen bg-black text-white">
      <nav className="border-b border-white/10 px-6 py-4">
        <Link href="/" className="flex items-center gap-2 font-bold text-xl">
          <Sparkles className="h-6 w-6 text-purple-400" />
          <span>Noracan</span>
        </Link>
      </nav>

      <article className="max-w-3xl mx-auto px-6 py-16 prose prose-invert">
        <h1 className="text-4xl font-bold mb-2">Terms of Service</h1>
        <p className="text-gray-400 mb-12">Last updated: July 4, 2026</p>

        <section className="space-y-6 text-gray-300">
          <div>
            <h2 className="text-xl font-semibold text-white mb-2">1. Acceptance of Terms</h2>
            <p>By using Noracan, you agree to these terms. If you don't agree, don't use the service.</p>
          </div>

          <div>
            <h2 className="text-xl font-semibold text-white mb-2">2. Beta Access</h2>
            <p>During the beta period, Noracan is provided free of charge. Features may change. We may terminate beta access at any time with reasonable notice.</p>
          </div>

          <div>
            <h2 className="text-xl font-semibold text-white mb-2">3. Your Responsibilities</h2>
            <p>You're responsible for: keeping your account credentials secure, ensuring your workflows comply with applicable laws, and respecting rate limits of connected services.</p>
          </div>

          <div>
            <h2 className="text-xl font-semibold text-white mb-2">4. Acceptable Use</h2>
            <p>You may not use Noracan for: spam, harassment, illegal activities, or to violate third-party terms of service.</p>
          </div>

          <div>
            <h2 className="text-xl font-semibold text-white mb-2">5. Intellectual Property</h2>
            <p>Noracan and its original content are protected by copyright. Your workflows and data remain yours.</p>
          </div>

          <div>
            <h2 className="text-xl font-semibold text-white mb-2">6. Limitation of Liability</h2>
            <p>Noracan is provided "as is" during beta. We're not liable for any damages arising from your use of the service.</p>
          </div>

          <div>
            <h2 className="text-xl font-semibold text-white mb-2">7. Changes to Terms</h2>
            <p>We may update these terms. Continued use after changes constitutes acceptance.</p>
          </div>

          <div>
            <h2 className="text-xl font-semibold text-white mb-2">8. Contact</h2>
            <p>Questions? Email legal@noracan.app.</p>
          </div>
        </section>
      </article>
    </div>
  )
}

<footer className="border-t border-white/10 py-12 mt-20">
  <div className="container mx-auto px-6">
    <div className="flex flex-col md:flex-row justify-between items-center gap-4">
      <p className="text-gray-500 text-sm">
        &copy; {new Date().getFullYear()} Noracan. All rights reserved.
      </p>
      <div className="flex gap-6 text-sm text-gray-500">
        <Link href="/privacy" className="hover:text-white transition">Privacy</Link>
        <Link href="/terms" className="hover:text-white transition">Terms</Link>
        <Link href="/beta" className="hover:text-white transition">Beta</Link>
      </div>
    </div>
  </div>
</footer>

# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key

# Database
DATABASE_URL=postgresql://postgres:password@db.your-project.supabase.co:5432/postgres

# OpenAI
OPENAI_API_KEY=sk-...

# Analytics
NEXT_PUBLIC_POSTHOG_KEY=phc_your-key
NEXT_PUBLIC_POSTHOG_HOST=https://us.i.posthog.com

# Sentry (optional)
NEXT_PUBLIC_SENTRY_DSN=https://...@sentry.io/...

# OAuth (optional)
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
SLACK_CLIENT_ID=...
SLACK_CLIENT_SECRET=...

CREATE TABLE beta_signups (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  email TEXT NOT NULL UNIQUE,
  source TEXT DEFAULT 'beta_page',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE beta_signups ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Allow public signup" ON beta_signups
  FOR INSERT TO anon
  WITH CHECK (true);

CREATE POLICY "Service role can read" ON beta_signups
  FOR SELECT TO service_role
  USING (true);

# 1. Install packages
npm install posthog-js react-cookie-consent

# 2. Create the files above

# 3. Run SQL in Supabase

# 4. Add environment variables to Vercel

# 5. Deploy
git add .
git commit -m "Launch: beta page, analytics, legal pages"
git push


