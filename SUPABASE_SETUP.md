# 🚀 Supabase Setup Guide for Tumblera

## Complete Guide to Setting Up Your Database & Authentication

---

## Step 1: Create Supabase Project (5 minutes)

### 1.1 Sign Up
1. Go to [https://supabase.com](https://supabase.com)
2. Click **"Start your project"**
3. Sign up with GitHub (recommended) or email
4. Verify your email if needed

### 1.2 Create New Project
1. Click **"New Project"**
2. Fill in the details:
   - **Name:** Tumblera
   - **Database Password:** Create a strong password (save it somewhere safe!)
   - **Region:** Choose closest to your location
   - **Pricing Plan:** Free (perfect for starting!)
3. Click **"Create new project"**
4. Wait 2-3 minutes for project to be ready

---

## Step 2: Get Your API Keys (2 minutes)

1. Once project is ready, click **"Settings"** (gear icon in sidebar)
2. Go to **"API"** section
3. You'll see two important values:
   - **Project URL** (looks like: `https://xxxxx.supabase.co`)
   - **anon/public key** (long string starting with `eyJ...`)
4. **Copy both values** - you'll need them soon!

---

## Step 3: Create Database Tables (5 minutes)

### 3.1 Create Users Table
1. Go to **"SQL Editor"** in the sidebar
2. Click **"New Query"**
3. Paste this SQL code:

```sql
-- Create users table
CREATE TABLE users (
    id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
    email TEXT UNIQUE NOT NULL,
    first_name TEXT,
    last_name TEXT,
    phone TEXT,
    role TEXT DEFAULT 'customer' CHECK (role IN ('customer', 'seller', 'admin')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY "Users can view own profile"
    ON users FOR SELECT
    USING (auth.uid() = id);

CREATE POLICY "Users can update own profile"
    ON users FOR UPDATE
    USING (auth.uid() = id);

-- Sellers can view all users
CREATE POLICY "Sellers can view all users"
    ON users FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM users
            WHERE id = auth.uid() AND role IN ('seller', 'admin')
        )
    );
```

4. Click **"Run"** (or press F5)
5. You should see ✅ **"Success. No rows returned"**

### 3.2 Create Orders Table
1. Click **"New Query"** again
2. Paste this SQL code:

```sql
-- Create orders table
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES auth.users(id) ON DELETE SET NULL,
    customer_name TEXT NOT NULL,
    customer_email TEXT NOT NULL,
    customer_phone TEXT NOT NULL,
    customer_address TEXT NOT NULL,
    customer_notes TEXT,
    items JSONB NOT NULL,
    subtotal DECIMAL(10,2) NOT NULL,
    shipping DECIMAL(10,2) NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
    payment_method TEXT DEFAULT 'cash_on_delivery',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Create policies
-- Anyone can create orders (even guests)
CREATE POLICY "Anyone can create orders"
    ON orders FOR INSERT
    WITH CHECK (true);

-- Users can view their own orders
CREATE POLICY "Users can view own orders"
    ON orders FOR SELECT
    USING (auth.uid() = user_id);

-- Sellers can view all orders
CREATE POLICY "Sellers can view all orders"
    ON orders FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM users
            WHERE id = auth.uid() AND role IN ('seller', 'admin')
        )
    );

-- Sellers can update order status
CREATE POLICY "Sellers can update orders"
    ON orders FOR UPDATE
    USING (
        EXISTS (
            SELECT 1 FROM users
            WHERE id = auth.uid() AND role IN ('seller', 'admin')
        )
    );

-- Create index for faster queries
CREATE INDEX orders_user_id_idx ON orders(user_id);
CREATE INDEX orders_status_idx ON orders(status);
CREATE INDEX orders_created_at_idx ON orders(created_at DESC);
```

3. Click **"Run"**
4. You should see ✅ **"Success"**

---

## Step 4: Set Up Storage for Images (3 minutes)

### 4.1 Create Storage Bucket
1. Go to **"Storage"** in the sidebar
2. Click **"Create a new bucket"**
3. Fill in:
   - **Name:** `designs`
   - **Public bucket:** ✅ Check this (so images are publicly accessible)
4. Click **"Create bucket"**

### 4.2 Set Storage Policies
1. Click on the **"designs"** bucket
2. Go to **"Policies"** tab
3. Click **"New Policy"**
4. Select **"For full customization"**
5. Name it: "Public Access"
6. Paste this:

```sql
-- Allow anyone to upload
CREATE POLICY "Anyone can upload designs"
ON storage.objects FOR INSERT
WITH CHECK (bucket_id = 'designs');

-- Allow anyone to read/download
CREATE POLICY "Anyone can view designs"
ON storage.objects FOR SELECT
USING (bucket_id = 'designs');
```

---

## Step 5: Configure Your Website (2 minutes)

### 5.1 Update Supabase Configuration
1. Open your project folder
2. Go to `js/supabase.js`
3. Find these lines at the top:

```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

4. Replace with your actual values:
```javascript
const SUPABASE_URL = 'https://xxxxx.supabase.co'; // Your Project URL
const SUPABASE_ANON_KEY = 'eyJxxx...'; // Your anon/public key
```

5. **Save the file**

---

## Step 6: Create Your First Seller Account (3 minutes)

### 6.1 Sign Up Through Website
1. Open your website
2. Go to Sign Up page
3. Create an account with your email

### 6.2 Make Yourself a Seller
1. Go back to Supabase Dashboard
2. Click **"Table Editor"** in sidebar
3. Click **"users"** table
4. Find your newly created user row
5. Click the **"role"** cell
6. Change from `customer` to `seller`
7. Click ✅ to save

**Done! Now you can access the seller dashboard!**

---

## Step 7: Enable Email Authentication (Optional - 2 minutes)

### 7.1 Configure Email Settings
1. Go to **"Authentication"** → **"Settings"**
2. Under **"Email Auth"**, make sure it's enabled
3. Scroll to **"Email Templates"**
4. Customize confirmation email (optional)

### 7.2 For Production (Custom Domain)
1. Go to **"Settings"** → **"Auth"**
2. Add your site URL: `https://tumblera.me`
3. Add redirect URLs if needed

---

## Step 8: Test Everything (5 minutes)

### 8.1 Test Customer Features
- [ ] Sign up as a new customer
- [ ] Login with that account
- [ ] Create a custom tumbler
- [ ] Add to cart
- [ ] Complete checkout
- [ ] Order should appear in database

### 8.2 Test Seller Features
- [ ] Login as seller (the account you made seller in Step 6)
- [ ] Go to `/seller.html`
- [ ] You should see the dashboard
- [ ] The order you just created should appear
- [ ] Try changing order status
- [ ] Try searching/filtering orders

---

## 🎉 You're All Set!

Your Tumblera website now has:
- ✅ User authentication (sign up/login)
- ✅ Customer accounts
- ✅ Seller dashboard
- ✅ Order management
- ✅ Database storage
- ✅ Image uploads

---

## 📊 Supabase Free Tier Limits

Your free plan includes:
- **Database:** 500 MB
- **Storage:** 1 GB
- **Bandwidth:** 2 GB/month  
- **API Requests:** Unlimited
- **Auth Users:** Unlimited

This is perfect for starting out! Upgrade later if needed.

---

## 🔒 Security Best Practices

### Important Notes:
1. **Never commit your Supabase keys to Git if your repository is public**
   - The `anon` key is safe to use in frontend code
   - Keep your `service_role` key SECRET (we don't use it in this project)

2. **Row Level Security (RLS) is enabled**
   - Users can only see their own orders
   - Sellers can see all orders
   - Data is protected automatically

3. **For production:**
   - Set up custom email templates
   - Configure password requirements
   - Enable 2FA for seller accounts
   - Set up monitoring and alerts

---

## 🐛 Troubleshooting

### "Failed to save order"
- Check that your API keys are correct in `js/supabase.js`
- Verify tables were created successfully
- Check browser console for specific errors

### "Permission denied"
- Make sure RLS policies were created
- Check that user role is set correctly for sellers
- Verify you're logged in

### "Images not uploading"
- Check that storage bucket "designs" exists
- Verify storage policies are set
- Check file size (max 5MB in our config)

### "Can't access seller dashboard"
- Make sure your user account has role = 'seller' or 'admin'
- Check that you're logged in
- Clear browser cache and try again

---

## 📞 Need Help?

- **Supabase Docs:** https://supabase.com/docs
- **Supabase Discord:** https://discord.supabase.com
- **Supabase Status:** https://status.supabase.com

---

## 🚀 Next Steps

Now that your database is set up:

1. **Commit and push** your code to GitHub
2. **Set up custom domain** (see CUSTOM_DOMAIN.md)
3. **Test thoroughly** before sharing
4. **Create additional seller accounts** if needed
5. **Customize** the design to match your brand

---

*Last Updated: November 6, 2025*
