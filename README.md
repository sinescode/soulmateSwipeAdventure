**`app.py`**

```python
from flask import Flask, render_template, request, jsonify
from threading import Thread
from database import get_db_session, WithdrawalRequest, User
from sqlalchemy import or_
from datetime import datetime, timedelta, timezone

app = Flask(__name__)

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/unauthorised')
def unauthorised():
    return render_template('unauthorized.html')


@app.route('/api/user/balance', methods=['GET'])
def get_user_balance():
    """Get user balance by Telegram user ID"""
    user_id = request.args.get('user_id')
    if not user_id:
        return jsonify({'error': 'user_id parameter is required'}), 400
    try:
        user_id = int(user_id)
    except ValueError:
        return jsonify({'error': 'user_id must be an integer'}), 400

    session = get_db_session()
    try:
        user = session.query(User).filter(User.user_id == user_id).first()
        if not user:
            return jsonify({'error': 'User not found'}), 404

        return jsonify({
            'user_id': user.user_id,
            'balance': user.balance,
            'username': user.username,
            'first_name': user.first_name
        })
    except Exception as e:
        return jsonify({'error': str(e)}), 500
    finally:
        session.close()


@app.route('/api/user/add_points', methods=['POST'])
def add_points_to_user():
    """Add points to user balance after ad view"""
    data = request.get_json()
    if not data or 'user_id' not in data:
        return jsonify({'error': 'user_id is required'}), 400

    try:
        user_id = int(data['user_id'])
        points_to_add = int(data.get('points', 1))  # Default 1 point per ad
    except ValueError:
        return jsonify({'error': 'user_id and points must be integers'}), 400

    session = get_db_session()
    try:
        user = session.query(User).filter(User.user_id == user_id).first()
        if not user:
            return jsonify({'error': 'User not found'}), 404

        # Prevent duplicate rewards within 1 minute
        threshold = datetime.now(timezone.utc) - timedelta(minutes=1)
        if user.last_active and user.last_active > threshold:
            return jsonify({
                'success': True,
                'new_balance': user.balance,
                'points_added': 0,
                'message': 'Points already awarded recently'
            })

        # Update balance and earned total
        user.balance += points_to_add
        user.earned_total += points_to_add
        user.last_active = datetime.now(timezone.utc)

        session.commit()
        return jsonify({
            'success': True,
            'new_balance': user.balance,
            'points_added': points_to_add
        })
    except Exception as e:
        session.rollback()
        return jsonify({'error': str(e)}), 500
    finally:
        session.close()


@app.route('/withdrawals')
def withdrawals():
    status_filter = request.args.get('status', 'pending')
    user_request = request.args.get('user_request', '')

    session = get_db_session()
    try:
        query = session.query(WithdrawalRequest)

        if status_filter:
            query = query.filter(WithdrawalRequest.status == status_filter)

        if user_request:
            try:
                request_id = int(user_request)
                query = query.filter(or_(
                    WithdrawalRequest.user_id == user_request,
                    WithdrawalRequest.id == request_id
                ))
            except ValueError:
                query = query.filter(WithdrawalRequest.user_id == user_request)

        withdrawals = query.order_by(WithdrawalRequest.created_at.desc()).all()

        pending_count = session.query(WithdrawalRequest).filter_by(status='pending').count()
        approved_count = session.query(WithdrawalRequest).filter_by(status='approved').count()
        rejected_count = session.query(WithdrawalRequest).filter_by(status='rejected').count()

    finally:
        session.close()

    return render_template('withdrawals.html',
                           withdrawals=withdrawals,
                           status_filter=status_filter,
                           user_request=user_request,
                           pending_count=pending_count,
                           approved_count=approved_count,
                           rejected_count=rejected_count,
                           timedelta=timedelta)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=10000, debug=True)
```

---

**`templates/index.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Telegram Mini App</title>
    <!-- Telegram Web App SDK -->
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <!-- RichAds Integration -->
    <script src="https://richinfo.co/richpartners/telegram/js/tg-ob.js"></script>
    <script>
        window.TelegramAdsController = new TelegramAdsController();
        window.TelegramAdsController.initialize({
            pubId: "970316",
            appId: "2175",
        });
    </script>
    <style>
        /* ... your existing CSS ... */
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Welcome to EarnApp</h1>
            <p class="subtitle">Watch ads and earn rewards directly in Telegram</p>
        </div>
        <div class="card">
            <p class="balance-label">YOUR BALANCE</p>
            <div class="balance-display" id="balance-display">Loading...</div>
        </div>
        <button id="watch-ad-button">🎥 Watch Ad & Earn Points</button>
        <a href="/withdrawals" class="withdrawals">💳 Withdraw Earnings</a>
        <button id="close-button">✕ Close App</button>
        <div class="user-data" id="user-data"></div>
        <div class="footer">
            <p>EarnApp © 2023 • All rights reserved</p>
        </div>
    </div>
    <script>
        function checkTelegramEnvironment() {
            if (!window.Telegram || !window.Telegram.WebApp) {
                redirectToUnauthorized();
                return false;
            }
            const tg = window.Telegram.WebApp;
            const user = tg.initDataUnsafe?.user;
            if (!user || !user.id) {
                redirectToUnauthorized();
                return false;
            }
            return true;
        }
        function redirectToUnauthorized() {
            if (!window.location.pathname.endsWith('/unauthorised')) {
                window.location.href = '/unauthorised';
            }
        }
        async function fetchUserBalance(userId) {
            const display = document.getElementById('balance-display');
            display.textContent = '...';
            try {
                const res = await fetch(`${window.location.origin}/api/user/balance?user_id=${userId}`);
                const data = await res.json();
                if (res.ok && !data.error) {
                    display.textContent = `${data.balance} pts`;
                } else {
                    throw new Error(data.error || 'Unknown error');
                }
            } catch (e) {
                display.textContent = 'Error';
                window.Telegram.WebApp.showAlert('Failed to load balance');
            }
        }
        function initializeApp() {
            if (!checkTelegramEnvironment()) return;
            const tg = window.Telegram.WebApp;
            tg.expand();
            const user = tg.initDataUnsafe.user;
            // Display user data
            document.getElementById('user-data').innerHTML = `
                <h2>Your Account</h2>
                <div class="data-row"><span class="data-label">Telegram ID:</span><span class="data-value">${user.id}</span></div>
                <div class="data-row"><span class="data-label">Name:</span><span class="data-value">${user.first_name} ${user.last_name || ''}</span></div>
                ${user.username ? `<div class="data-row"><span class="data-label">Username:</span><span class="data-value">@${user.username}</span></div>` : ''}
            `;
            fetchUserBalance(user.id);

            document.getElementById('watch-ad-button').addEventListener('click', async () => {
                const btn = document.getElementById('watch-ad-button');
                const orig = btn.textContent;
                btn.innerHTML = '<span class="spinner"></span> Loading Ad...';
                btn.disabled = true;
                let completed = false;
                try {
                    completed = await new Promise(resolve => {
                        let finished = false;
                        let started = false;
                        const t = setTimeout(() => !finished && resolve(false), 60000);
                        try {
                            window.TelegramAdsController.showAd({
                                onAdLoaded: () => started = true,
                                onRewarded: () => { finished = true; clearTimeout(t); resolve(true); },
                                onAdClosed: () => { finished = true; clearTimeout(t); resolve(false); },
                                onError: () => { finished = true; clearTimeout(t); resolve(false); }
                            });
                        } catch (_) { clearTimeout(t); resolve(false); }
                        setTimeout(() => !started && !finished && (finished = true, clearTimeout(t), resolve(false)), 3000);
                    });
                    if (!completed) {
                        tg.showAlert('Please watch the full ad to earn points');
                        return;
                    }
                    btn.innerHTML = '<span class="spinner"></span> Adding Points...';
                    const res = await fetch(`${window.location.origin}/api/user/add_points`, {
                        method: 'POST',
                        headers: {'Content-Type': 'application/json'},
                        body: JSON.stringify({ user_id: user.id, points: 1 })
                    });
                    const data = await res.json();
                    if (!res.ok || data.error) throw new Error(data.error || 'Failed to add points');
                    fetchUserBalance(user.id);
                    tg.showPopup({ title: 'Success!', message: `You earned ${data.points_added} point(s)! New balance: ${data.new_balance}`, buttons: [{ type: 'ok' }] });
                } catch (e) {
                    console.error(e);
                    tg.showAlert('Error processing ad: ' + e.message);
                } finally {
                    btn.textContent = orig;
                    btn.disabled = false;
                }
            });

            document.getElementById('close-button').addEventListener('click', () => tg.close());
            tg.onEvent('themeChanged', () => {
                document.body.style.backgroundColor = tg.themeParams.bg_color;
                document.body.style.color = tg.themeParams.text_color;
            });
        }
        initializeApp();
    </script>
</body>
</html>
```

