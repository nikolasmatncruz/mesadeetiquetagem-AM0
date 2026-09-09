<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mesa de Etiquetagem — Painel de Controle</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<style>
  :root{
    --navy:#1F3864;
    --navy-dark:#0F1E3D;
    --navy-soft:#2E4C82;
    --green:#6AA84F;
    --green-light:#8FCB6C;
    --green-dark:#38761D;
    --green-deep:#20460F;
    --gold:#D9A521;
    --gold-light:#F2C94C;
    --bg-canvas:#E3E8EE;
    --white:#FFFFFF;
    --text:#1B2130;
    --text-mute:#66728A;
    --text-faint:#93A0B5;
    --danger:#C0392B;
    --border:rgba(27,33,48,.08);
    --glass:rgba(255,255,255,.6);
    --glass-strong:rgba(255,255,255,.82);
    --glass-border:rgba(255,255,255,.7);
    --glass-green:rgba(255,255,255,.28);
    --radius-xl:26px;
    --radius-lg:18px;
    --radius-md:14px;
    --radius-sm:10px;
    --shadow-sm:0 1px 2px rgba(20,30,55,.06);
    --shadow-md:0 8px 24px rgba(20,30,55,.10);
    --shadow-lg:0 24px 60px rgba(15,25,50,.22);
  }
  *{box-sizing:border-box;}
  html,body{height:100%;}
  body{
    margin:0;
    font-family:"Segoe UI", Arial, Helvetica, sans-serif;
    background:
      radial-gradient(circle at 15% 10%, #eef3ea 0%, transparent 45%),
      radial-gradient(circle at 85% 90%, #e4ecf6 0%, transparent 45%),
      var(--bg-canvas);
    color:var(--text);
    -webkit-font-smoothing:antialiased;
    font-size:14px;
    display:flex; align-items:center; justify-content:center;
    min-height:100vh;
    padding:22px;
  }
  .num{font-variant-numeric:tabular-nums;}
  button, input{font-family:inherit;}

  /* ================= SHELL ================= */
  .shell{
    width:100%; max-width:1660px; height:min(960px, 94vh);
    background:var(--white);
    border-radius:var(--radius-xl);
    box-shadow:var(--shadow-lg);
    display:flex; overflow:hidden;
    position:relative;
  }

  /* ================= LEFT SIDEBAR ================= */
  .side-left{
    width:220px; flex-shrink:0;
    background:linear-gradient(165deg, var(--green) 0%, var(--green-dark) 65%, var(--green-deep) 100%);
    position:relative; overflow:hidden;
    display:flex; flex-direction:column;
    padding:26px 16px 18px;
  }
  .side-left .blob{
    position:absolute; border-radius:50%; filter:blur(38px); opacity:.35; background:#fff;
  }
  .side-left .blob.b1{width:180px; height:180px; top:-60px; left:-60px;}
  .side-left .blob.b2{width:140px; height:140px; bottom:60px; right:-50px; background:var(--gold-light); opacity:.25;}

  .brand-mark{
    display:flex; flex-direction:column; align-items:center; gap:2px;
    margin-bottom:22px; position:relative; z-index:2;
  }
  .brand-mark .logo-badge{
    width:112px; height:112px; object-fit:contain;
    filter:drop-shadow(0 6px 14px rgba(0,0,0,.28));
  }

  .nav-scroll{flex:1; overflow-y:auto; position:relative; z-index:2; padding-right:2px;}
  .nav-label{color:rgba(255,255,255,.65); font-size:9.5px; font-weight:700; text-transform:uppercase; letter-spacing:1px; margin:16px 8px 8px;}
  .nav-label:first-child{margin-top:0;}
  .nav-item{
    display:flex; align-items:center; gap:10px;
    padding:10px 12px; border-radius:var(--radius-sm); cursor:pointer;
    color:rgba(255,255,255,.88); font-size:12.6px; font-weight:600; margin-bottom:3px;
    transition:background .15s ease, color .15s ease;
    white-space:nowrap; overflow:hidden;
  }
  .nav-item .ic{width:15px; text-align:center; flex-shrink:0; font-size:13px;}
  .nav-item .lbl{overflow:hidden; text-overflow:ellipsis; flex:1;}
  .nav-item .rm{opacity:0; font-size:14px; color:#FFD9D2;}
  .nav-item:hover .rm{opacity:.8;}
  .nav-item:hover{background:rgba(255,255,255,.14);}
  .nav-item.active{
    background:var(--glass-strong);
    color:var(--green-deep);
    box-shadow:var(--shadow-sm);
    font-weight:800;
  }
  .nav-item.active .rm{color:var(--danger);}

  .add-day-btn{
    margin-top:12px; padding:10px; border-radius:var(--radius-sm);
    border:1.5px dashed rgba(255,255,255,.55);
    background:rgba(255,255,255,.10);
    backdrop-filter:blur(6px);
    color:#fff; font-weight:700; font-size:11.5px; text-align:center; cursor:pointer;
    position:relative; z-index:2;
  }
  .add-day-btn:hover{background:rgba(255,255,255,.2);}

  /* ================= MAIN ================= */
  .main{flex:1; min-width:0; overflow-y:auto; background:#F7F8F7;}
  .main-inner{padding:24px 26px 60px;}
  .main-inner.view-in{animation:fadeInUp .28s ease;}
  @keyframes fadeInUp{
    from{opacity:0; transform:translateY(6px);}
    to{opacity:1; transform:translateY(0);}
  }

  .top-header{display:flex; align-items:center; justify-content:space-between; gap:16px; margin-bottom:18px; flex-wrap:wrap;}
  .top-header h2{font-size:23px; font-weight:800; color:var(--navy); margin:0;}
  .top-header .datepill{font-size:12px; color:var(--text-mute); margin-top:2px;}
  .header-actions{display:flex; align-items:center; gap:10px;}
  .search-wrap{position:relative; display:flex; align-items:center;}
  .search-wrap .search-ic{position:absolute; left:14px; color:var(--text-faint); display:flex; pointer-events:none;}
  .glass-input{
    background:var(--glass); backdrop-filter:blur(10px); -webkit-backdrop-filter:blur(10px);
    border:1px solid var(--glass-border); border-radius:24px;
    padding:9px 16px 9px 38px; font-size:12.5px; color:var(--text); width:190px;
    box-shadow:var(--shadow-sm);
  }
  .glass-input::placeholder{color:var(--text-faint);}
  .glass-input:focus{outline:none; border-color:var(--green);}
  .search-wrap:focus-within .search-ic{color:var(--green-dark);}
  .icon-btn{
    width:38px; height:38px; border-radius:50%;
    background:var(--glass); backdrop-filter:blur(10px); -webkit-backdrop-filter:blur(10px);
    border:1px solid var(--glass-border); box-shadow:var(--shadow-sm);
    display:flex; align-items:center; justify-content:center; font-size:14px; color:var(--navy);
    cursor:pointer; flex-shrink:0;
  }
  .glass-select{
    background:var(--glass); backdrop-filter:blur(10px); -webkit-backdrop-filter:blur(10px);
    border:1px solid var(--glass-border); border-radius:20px;
    padding:9px 12px; font-size:12px; color:var(--text); box-shadow:var(--shadow-sm);
    cursor:pointer;
  }
  .glass-select:focus{outline:none; border-color:var(--green);}
  .dl-btn{
    display:inline-flex; align-items:center; gap:7px;
    background:var(--navy); color:#fff; border:none;
    padding:9px 16px 9px 13px; border-radius:20px; font-size:12px; font-weight:700; cursor:pointer;
    box-shadow:var(--shadow-sm); white-space:nowrap; flex-shrink:0;
  }
  .dl-btn:hover{background:var(--navy-soft); transform:translateY(-1px); box-shadow:var(--shadow-md);}
  .dl-btn:active{transform:translateY(1px); box-shadow:var(--shadow-sm);}
  .dl-btn svg{flex-shrink:0;}
  .live-badge{
    display:flex; align-items:center; gap:6px;
    background:var(--glass); backdrop-filter:blur(10px);
    border:1px solid var(--glass-border); border-radius:20px; padding:7px 13px 7px 10px;
    font-size:11px; color:var(--green-dark); font-weight:700; box-shadow:var(--shadow-sm);
  }
  .live-dot{width:7px; height:7px; border-radius:50%; background:var(--green); box-shadow:0 0 0 3px rgba(106,168,79,.25);}

  /* ---- hero banner ---- */
  .hero{
    position:relative; overflow:hidden;
    border-radius:var(--radius-lg);
    background:linear-gradient(120deg, #DCEED3 0%, #C9E6C4 100%);
    padding:26px 30px; margin-bottom:20px;
    display:flex; align-items:center; justify-content:space-between; gap:20px;
  }
  .hero .blob{position:absolute; border-radius:50%; filter:blur(30px);}
  .hero .blob1{width:220px; height:220px; background:rgba(255,255,255,.55); top:-90px; right:120px;}
  .hero .blob2{width:160px; height:160px; background:var(--gold-light); opacity:.35; bottom:-70px; right:-30px;}
  .hero-text{position:relative; z-index:2;}
  .hero-text h3{margin:0; font-size:22px; color:var(--navy); font-weight:800;}
  .hero-text p{margin:6px 0 0; font-size:13px; color:var(--green-deep); max-width:360px;}
  .hero-cta{
    margin-top:14px; display:inline-flex; align-items:center; gap:7px;
    background:var(--navy); color:#fff; border:none; padding:10px 18px 10px 15px; border-radius:22px;
    font-size:12.5px; font-weight:700; cursor:pointer; box-shadow:var(--shadow-sm);
  }
  .hero-cta svg{flex-shrink:0;}
  .hero-cta:hover{background:var(--navy-soft);}
  .hero-illustration{position:relative; z-index:2; flex-shrink:0;}

  /* ---- grid: activity + stat card ---- */
  .row-2col{display:grid; grid-template-columns:1.35fr 1fr; gap:18px; margin-bottom:18px;}
  .panel{
    background:var(--white); border:1px solid var(--border); border-radius:var(--radius-lg);
    padding:20px; box-shadow:var(--shadow-sm);
    transition:box-shadow .18s ease, transform .18s ease;
  }
  .panel:hover{box-shadow:var(--shadow-md); transform:translateY(-1px);}
  .panel-head{display:flex; align-items:center; justify-content:space-between; margin-bottom:16px;}
  .panel-head h4{margin:0; font-size:14.5px; font-weight:800; color:var(--navy); display:flex; align-items:center; gap:8px;}
  .panel-head .tag{font-size:11px; color:var(--text-mute); background:#F1F3F0; padding:5px 11px; border-radius:14px; font-weight:600;}

  .capsules{display:flex; align-items:flex-end; justify-content:space-between; gap:10px; height:170px; padding:0 4px;}
  .capsule-col{display:flex; flex-direction:column; align-items:center; gap:6px; flex:1; height:100%; justify-content:flex-end; cursor:default;}
  .capsule-value{font-size:9.5px; font-weight:800; color:var(--navy); font-variant-numeric:tabular-nums;}
  .capsule-track{
    width:14px; flex:1; background:#EDF0EC; border-radius:9px; position:relative; overflow:hidden;
    display:flex; align-items:flex-end;
  }
  .capsule-fill{width:100%; border-radius:9px; transition:height .3s ease, filter .15s ease;}
  .capsule-col:hover .capsule-fill{filter:brightness(1.12);}
  .capsule-col:hover .capsule-value{color:var(--green-dark);}
  .capsule-day{font-size:9.5px; color:var(--text-mute); font-weight:700; text-align:center; max-width:46px; overflow:hidden; text-overflow:ellipsis; white-space:nowrap;}
  .legend-row{display:flex; gap:18px; margin-top:14px; justify-content:center; flex-wrap:wrap;}
  .legend-item{display:flex; align-items:center; gap:6px; font-size:11px; color:var(--text-mute); font-weight:600;}
  .legend-dot{width:8px; height:8px; border-radius:50%;}

  .stat-glass{
    border-radius:var(--radius-lg); padding:22px; position:relative; overflow:hidden; color:#fff;
    background:linear-gradient(160deg, var(--green) 0%, var(--green-dark) 55%, var(--green-deep) 100%);
    display:flex; flex-direction:column; height:100%;
    box-shadow:var(--shadow-md);
  }
  .stat-glass .deco{position:absolute; border-radius:50%; background:rgba(255,255,255,.14); filter:blur(4px);}
  .stat-glass .deco1{width:150px; height:150px; top:-50px; right:-40px;}
  .stat-glass .head{display:flex; align-items:center; justify-content:space-between; position:relative; z-index:2;}
  .stat-glass .head span{font-size:11.5px; font-weight:700; opacity:.85;}
  .stat-glass .pill{background:rgba(255,255,255,.18); backdrop-filter:blur(6px); font-size:10.5px; padding:4px 10px; border-radius:12px;}
  .stat-glass .big{font-size:34px; font-weight:800; margin-top:10px; position:relative; z-index:2;}
  .stat-glass .big small{font-size:13px; font-weight:600; opacity:.8; margin-left:4px;}
  .stat-glass .chart-wrap{flex:1; margin-top:8px; position:relative; z-index:2; min-height:90px;}
  .stat-glass .footnote{display:flex; justify-content:space-between; font-size:9.5px; opacity:.7; margin-top:6px; position:relative; z-index:2;}

  /* ---- mini stat cards row ---- */
  .mini-row{display:grid; grid-template-columns:repeat(3,1fr); gap:16px; margin-bottom:26px;}
  .mini-card{
    border-radius:var(--radius-lg); padding:16px 18px; color:#fff; position:relative; overflow:hidden;
    box-shadow:var(--shadow-md); transition:transform .18s ease, box-shadow .18s ease;
  }
  .mini-card:hover{transform:translateY(-2px); box-shadow:var(--shadow-lg);}
  .mini-card.c1{background:linear-gradient(135deg, var(--green) 0%, var(--green-dark) 100%);}
  .mini-card.c2{background:linear-gradient(135deg, var(--navy-soft) 0%, var(--navy) 100%);}
  .mini-card.c3{background:linear-gradient(135deg, var(--navy) 0%, var(--navy-dark) 100%);}
  .mini-card .deco{position:absolute; inset:0; background:radial-gradient(circle at 90% -10%, rgba(255,255,255,.18), transparent 55%);}
  .mini-card .top{display:flex; align-items:center; justify-content:space-between; position:relative; z-index:2;}
  .mini-card .top .lbl{display:flex; align-items:center; gap:8px; font-size:12px; font-weight:700;}
  .mini-ic{
    width:24px; height:24px; border-radius:8px; background:rgba(255,255,255,.20);
    display:flex; align-items:center; justify-content:center; flex-shrink:0;
  }
  .mini-card .delta{font-size:10.5px; font-weight:700; background:rgba(255,255,255,.18); padding:3px 8px; border-radius:10px;}
  .mini-card .val{font-size:22px; font-weight:800; margin-top:10px; position:relative; z-index:2;}
  .mini-card .val small{font-size:11.5px; font-weight:600; opacity:.75;}
  .mini-card .spark{height:34px; margin-top:6px; position:relative; z-index:2;}

  /* ---- analysis section (detailed charts, kept from before, restyled) ---- */
  .section-title{
    font-size:11.5px; font-weight:800; color:var(--navy); text-transform:uppercase; letter-spacing:.7px;
    margin:26px 0 14px; display:flex; align-items:center; gap:10px;
  }
  .section-title::after{content:""; flex:1; height:1px; background:var(--border);}
  .chart-grid{display:grid; grid-template-columns:repeat(3,1fr); gap:16px;}
  .card{background:var(--white); border:1px solid var(--border); border-radius:var(--radius-lg); padding:18px; box-shadow:var(--shadow-sm); transition:box-shadow .18s ease, transform .18s ease;}
  .card:hover{box-shadow:var(--shadow-md); transform:translateY(-1px);}
  .card.table-card:hover{transform:none;}
  .card h3{margin:0; font-size:13px; color:var(--navy); font-weight:800; display:flex; align-items:center; gap:7px;}
  .h-ic{
    display:flex; align-items:center; justify-content:center;
    width:22px; height:22px; border-radius:7px; flex-shrink:0;
    background:#E5F1E0; color:var(--green-dark);
  }
  .card .chart-sub{font-size:10.5px; color:var(--text-mute); margin:2px 0 12px;}
  .chart-wrap{position:relative; height:220px;}
  .chart-offline{
    height:100%; min-height:90px; display:flex; align-items:center; justify-content:center; gap:8px;
    text-align:center; font-size:11px; color:var(--text-faint); background:#F7F8F7; border-radius:10px;
    padding:10px 14px; border:1px dashed var(--border-strong);
  }
  .chart-offline-ic{display:flex; flex-shrink:0;}
  .spark-offline{height:100%; border-radius:6px; background:rgba(255,255,255,.10);}
  .stat-glass .chart-offline{background:rgba(255,255,255,.12); color:rgba(255,255,255,.8); border-color:rgba(255,255,255,.3);}

  .table-card{padding:0; overflow:hidden;}
  .table-scroll{overflow-x:auto;}
  table{width:100%; border-collapse:collapse; font-size:12.2px;}
  thead th{background:var(--navy); color:#fff; font-size:10px; text-transform:uppercase; letter-spacing:.4px; padding:11px 10px; font-weight:700;}
  tbody td{padding:9px 10px; text-align:center; border-bottom:1px solid var(--border);}
  tbody tr:nth-child(even){background:#FAFBFA;}
  tbody tr:hover{background:#EEF5EC;}
  tbody td.abacell{font-weight:700; color:var(--navy); cursor:pointer; text-align:left;}
  tbody td.abacell:hover{color:var(--green-dark); text-decoration:underline;}
  tfoot td{background:#F1F3F0; color:var(--navy); font-weight:800; padding:11px 10px; border-top:2px solid #D9DEDA;}
  .trend-up{color:var(--green-dark); font-weight:700;}
  .trend-down{color:var(--danger); font-weight:700;}
  .badge-empty{font-size:10.5px; color:var(--text-faint); font-style:italic;}

  /* ================= RIGHT SIDEBAR ================= */
  .side-right{
    width:290px; flex-shrink:0;
    background:linear-gradient(180deg, #E9F3E5 0%, #DCEBD6 100%);
    padding:22px 20px; overflow-y:auto;
    position:relative;
  }
  .sr-top{display:flex; align-items:center; justify-content:flex-end; gap:10px; margin-bottom:18px;}
  .sr-icon{
    width:34px; height:34px; border-radius:50%; background:var(--glass-strong); backdrop-filter:blur(8px);
    border:1px solid var(--glass-border); display:flex; align-items:center; justify-content:center;
    font-size:13px; color:var(--navy); position:relative; box-shadow:var(--shadow-sm);
  }
  .sr-icon .badge{
    position:absolute; top:-3px; right:-3px; background:var(--danger); color:#fff; font-size:9px;
    border-radius:50%; width:16px; height:16px; display:flex; align-items:center; justify-content:center; font-weight:700;
  }

  .profile-card{
    background:var(--glass-strong); backdrop-filter:blur(12px); -webkit-backdrop-filter:blur(12px);
    border:1px solid var(--glass-border); border-radius:var(--radius-lg);
    padding:20px 16px; text-align:center; box-shadow:var(--shadow-sm); margin-bottom:18px;
  }
  .avatar-ring{
    width:66px; height:66px; border-radius:50%; margin:0 auto 10px;
    background:linear-gradient(135deg, var(--green) 0%, var(--navy) 100%);
    display:flex; align-items:center; justify-content:center; color:#fff; font-weight:800; font-size:20px;
    box-shadow:0 0 0 4px rgba(255,255,255,.6);
  }
  .profile-card .pname{font-size:14.5px; font-weight:800; color:var(--navy);}
  .profile-card .prole{font-size:11px; color:var(--green-dark); font-weight:700; margin-top:2px;}
  .profile-trio{display:flex; justify-content:space-between; margin-top:14px; padding-top:14px; border-top:1px solid rgba(31,56,100,.1);}
  .profile-trio .item{text-align:center; flex:1;}
  .profile-trio .item b{display:block; font-size:14px; color:var(--navy); font-weight:800;}
  .profile-trio .item span{font-size:9.5px; color:var(--text-mute); text-transform:uppercase; letter-spacing:.3px;}

  .goals-block{margin-bottom:20px;}
  .goals-block h5, .rank-block h5{display:flex; align-items:center; gap:7px;}
  .h5-ic{color:var(--green-dark); display:flex;}
  .goals-block h5{font-size:11.5px; text-transform:uppercase; letter-spacing:.6px; color:var(--navy); font-weight:800; margin:0 0 12px;}
  .goal-row{margin-bottom:14px;}
  .goal-row .g-top{display:flex; justify-content:space-between; align-items:center; font-size:11.5px; margin-bottom:6px;}
  .goal-row .g-top .g-name{font-weight:700; color:var(--text);}
  .goal-row .g-top .g-val{color:var(--text-mute); display:flex; align-items:center; gap:4px;}
  .goal-row .g-top input.g-target{
    width:56px; border:1px solid rgba(31,56,100,.18); border-radius:6px; padding:3px 5px; font-size:10.5px;
    background:rgba(255,255,255,.7); text-align:center;
  }
  .goal-track{height:8px; border-radius:5px; background:rgba(31,56,100,.10); overflow:hidden;}
  .goal-fill{height:100%; border-radius:5px; background:linear-gradient(90deg, var(--green) 0%, var(--green-dark) 100%);}

  .rank-block h5{font-size:11.5px; text-transform:uppercase; letter-spacing:.6px; color:var(--navy); font-weight:800; margin:0 0 12px;}
  .rank-row{
    display:flex; align-items:center; gap:10px;
    background:var(--glass-strong); backdrop-filter:blur(8px);
    border:1px solid var(--glass-border); border-radius:12px; padding:9px 12px; margin-bottom:8px;
    box-shadow:var(--shadow-sm);
  }
  .rank-row .pos{
    width:22px; height:22px; border-radius:50%; background:var(--navy); color:#fff; font-size:10.5px; font-weight:800;
    display:flex; align-items:center; justify-content:center; flex-shrink:0;
  }
  .rank-row:nth-child(2) .pos{background:var(--green-dark);}
  .rank-row .rname{flex:1; font-size:11.5px; font-weight:700; color:var(--text); overflow:hidden; text-overflow:ellipsis; white-space:nowrap;}
  .rank-row .rval{font-size:11.5px; font-weight:800; color:var(--navy);}
  .rank-empty{font-size:11px; color:var(--text-mute); font-style:italic; padding:6px 2px;}

  /* ================= DAY EDITOR ================= */
  .day-header{
    position:relative; overflow:hidden;
    background:linear-gradient(120deg, var(--navy) 0%, var(--navy-soft) 100%);
    color:#fff; border-radius:var(--radius-lg) var(--radius-lg) 0 0;
    padding:22px 26px; display:flex; align-items:center; justify-content:space-between; gap:16px; flex-wrap:wrap;
  }
  .day-header .deco{position:absolute; border-radius:50%; background:rgba(255,255,255,.10); filter:blur(2px);}
  .day-header .deco1{width:180px; height:180px; top:-70px; right:60px;}
  .day-header .name-block{display:flex; flex-direction:column; gap:4px; position:relative; z-index:2;}
  .day-header .eyebrow{font-size:10px; text-transform:uppercase; letter-spacing:.6px; color:#B7C6E4;}
  .day-header input.title-input{
    background:transparent; border:none; border-bottom:2px solid rgba(255,255,255,.28);
    color:#fff; font-size:20px; font-weight:800; padding:2px 2px 5px; width:360px; max-width:60vw; font-family:inherit;
  }
  .day-header input.title-input:focus{outline:none; border-bottom-color:var(--gold);}
  .period-field{display:flex; align-items:center; gap:8px; font-size:11px; color:#B7C6E4; font-weight:600; letter-spacing:.3px; text-transform:uppercase; position:relative; z-index:2;}
  .period-field input{
    background:rgba(255,255,255,.10); border:1px solid rgba(255,255,255,.28); border-radius:8px;
    color:#fff; font-size:13px; padding:7px 10px; width:160px; font-family:inherit;
  }
  .period-field input::placeholder{color:#8CA0C8;}
  .period-field input:focus{outline:none; border-color:var(--gold);}

  .day-body{background:var(--white); border:1px solid var(--border); border-top:none; border-radius:0 0 var(--radius-lg) var(--radius-lg); padding:24px; box-shadow:var(--shadow-sm);}

  .hours-table{width:100%; border-collapse:collapse; margin-bottom:16px;}
  .hours-table th{background:var(--navy); color:#fff; font-size:11px; font-weight:700; padding:11px 10px; text-align:center; border-bottom:3px solid var(--gold);}
  .hours-table th.timecol{text-align:left; padding-left:14px; width:200px;}
  .hours-table td{border:1px solid var(--border); padding:7px 8px; text-align:center;}
  .hours-table td.timecell{background:#F5F7F5; font-weight:700; color:var(--navy); text-align:left; padding-left:14px; font-size:12px;}
  .hours-table input.num{
    width:100%; border:1px solid rgba(31,56,100,.18); border-radius:6px; padding:8px 6px; text-align:center;
    font-size:13px; font-family:inherit; background:#FDF9EC; color:var(--text);
  }
  .hours-table input.num:focus{outline:none; border-color:var(--green); box-shadow:0 0 0 3px rgba(106,168,79,.18);}
  .hours-table td.hourtotal{background:#E5F1E0; font-weight:800; color:var(--navy);}
  .hours-table tfoot td{background:var(--green-dark); color:#fff; font-weight:800; padding:10px 8px; border-color:#2c5b16;}

  .team-name-row input.teamname{
    width:100%; border:1px solid rgba(31,56,100,.18); border-radius:6px; padding:8px 6px; text-align:center;
    font-size:12.5px; font-weight:700; color:var(--navy); background:#FDF9EC; font-family:inherit;
  }
  .team-name-row input.teamname:focus{outline:none; border-color:var(--green); box-shadow:0 0 0 3px rgba(106,168,79,.18);}

  .toolbar-row{display:flex; gap:10px; margin-bottom:22px; flex-wrap:wrap;}
  .pill-btn{border:1.4px dashed var(--green); background:#E5F1E0; color:var(--green-dark); font-weight:700; font-size:11.5px; padding:8px 14px; border-radius:20px; cursor:pointer;}
  .pill-btn:hover{background:#D9EAD3;}
  .pill-btn:disabled{opacity:.4; cursor:not-allowed;}
  .pill-btn.danger{border-color:#E5A79E; background:#FCEEEC; color:var(--danger);}
  .pill-btn.danger:hover{background:#FADCD8;}

  .summary-grid{display:grid; grid-template-columns:repeat(4,1fr); gap:14px;}
  .hour-goals{display:flex; flex-direction:column; gap:14px;}
  .goal-editor-row{
    display:flex; align-items:center; gap:8px; flex-wrap:wrap;
    font-size:12px; color:var(--text-mute); font-weight:600; margin:-6px 0 14px;
  }
  .goal-editor-row b{color:var(--navy); font-size:13px;}
  .goal-turno-input{
    width:84px; border:1px solid rgba(31,56,100,.18); border-radius:7px; padding:6px 8px;
    font-size:12.5px; font-weight:700; color:var(--navy); background:#FDF9EC; text-align:center; font-family:inherit;
  }
  .goal-turno-input:focus{outline:none; border-color:var(--green); box-shadow:0 0 0 3px rgba(106,168,79,.18);}
  .hour-goal-row{background:#F5F7F5; border:1px solid var(--border); border-radius:var(--radius-md); padding:14px 16px;}
  .hg-top{display:flex; align-items:baseline; justify-content:space-between; margin-bottom:9px;}
  .hg-label{font-size:11.5px; font-weight:800; color:var(--navy); text-transform:uppercase; letter-spacing:.4px;}
  .hg-val{font-size:15px; font-weight:800; color:var(--navy); font-variant-numeric:tabular-nums;}
  .hg-of{font-size:11px; font-weight:600; color:var(--text-mute);}
  .hg-track{height:9px; border-radius:6px; background:#E4E8E4; overflow:hidden;}
  .hg-fill{height:100%; border-radius:6px; transition:width .3s ease;}
  .hg-fill.hit{background:linear-gradient(90deg, var(--green-light), var(--green-dark));}
  .hg-fill.mid{background:linear-gradient(90deg, #F2C94C, var(--gold));}
  .hg-fill.miss{background:#D9DEDA;}
  .hg-fill.nogoal{background:#D9DEDA;}
  .hg-pct{font-size:10.5px; font-weight:700; margin-top:6px;}
  .hg-pct.hit{color:var(--green-dark);}
  .hg-pct.mid{color:#9C7A0E;}
  .hg-pct.miss, .hg-pct.nogoal{color:var(--text-faint); font-style:italic; font-weight:600;}
  .sum-card{background:#E5F1E0; border-radius:var(--radius-md); padding:14px 12px; text-align:center; border:1px solid #D9EAD3;}
  .sum-card .lbl{font-size:10px; font-weight:700; color:var(--green-dark); text-transform:uppercase; letter-spacing:.4px;}
  .sum-card .val{font-size:21px; font-weight:800; color:var(--navy); margin-top:6px;}
  .sum-card.small .val{font-size:17px;}
  .sum-card.warn{background:#FCEEEC; border-color:#F3C9C1;}
  .sum-card.warn .lbl{color:#A73224;}

  ::-webkit-scrollbar{width:9px; height:9px;}
  ::-webkit-scrollbar-thumb{background:#c3c9d3; border-radius:5px;}
  ::-webkit-scrollbar-track{background:transparent;}

  @media (max-width:980px){
    .row-2col{grid-template-columns:1fr;}
    .mini-row{grid-template-columns:1fr;}
    .chart-grid{grid-template-columns:1fr;}
  }
  @media (max-width:760px){
    body{padding:0;}
    .shell{border-radius:0; height:100vh;}
    .side-left{position:fixed; z-index:30; height:100%; box-shadow:var(--shadow-lg);}
  }
</style>
</head>
<body>
<div class="shell">

  <!-- LEFT NAV -->
  <div class="side-left">
    <div class="blob b1"></div><div class="blob b2"></div>
    <div class="brand-mark">
      <img class="logo-badge" src="data:image/png;base64,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" alt="Mesa de Etiquetagem">
    </div>
    <div class="nav-scroll">
      <div class="nav-label">Painel</div>
      <div id="nav-overview"></div>
      <div class="nav-label">Dias / Abas</div>
      <div id="nav-days"></div>
      <div class="add-day-btn" onclick="addDay()">＋ Novo dia</div>
      <div id="save-status" style="text-align:center; font-size:10.5px; color:rgba(255,255,255,.6); margin-top:8px;"></div>
    </div>
  </div>

  <!-- MAIN -->
  <div class="main"><div class="main-inner" id="main"></div></div>

  <!-- RIGHT SIDEBAR -->
  <div class="side-right" id="side-right"></div>
</div>

<script>
/* ======================= SUPABASE SYNC =======================
   Preencha SUPABASE_URL e SUPABASE_ANON_KEY depois de criar o projeto
   no Supabase (Project Settings > API). Enquanto estiverem vazios, o
   app funciona normalmente, só que sem salvar/carregar da nuvem. */
const SUPABASE_URL = 'https://aqznyfuarnsgkwcughnc.supabase.co';
const SUPABASE_ANON_KEY = 'sb_publishable__uYWGFFcWDGTrHMDCzgLEg_iU95PouW';
const SUPABASE_TABLE = 'mesa_etiquetagem';
const SUPABASE_ROW_ID = 'smg15'; // identifica o painel (permite reaproveitar a tabela para outras unidades no futuro)

let supabaseClient = null;
if (SUPABASE_URL && SUPABASE_ANON_KEY && window.supabase) {
  supabaseClient = window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
}

let saveTimer = null;
let saveStatusEl = null;
function setSaveStatus(text, isError){
  if (!saveStatusEl) saveStatusEl = document.getElementById('save-status');
  if (!saveStatusEl) return;
  saveStatusEl.textContent = text;
  saveStatusEl.style.color = isError ? '#C0392B' : 'rgba(255,255,255,.75)';
}

/* Debounced save: espera 1.2s de silêncio antes de gravar, para não
   disparar uma requisição a cada tecla digitada. */
function scheduleSave(){
  if (!supabaseClient) return;
  setSaveStatus('Salvando…');
  clearTimeout(saveTimer);
  saveTimer = setTimeout(saveStateToSupabase, 1200);
}

async function saveStateToSupabase(){
  if (!supabaseClient) return;
  try{
    const { error } = await supabaseClient
      .from(SUPABASE_TABLE)
      .upsert({ id: SUPABASE_ROW_ID, data: state, updated_at: new Date().toISOString() });
    if (error) throw error;
    setSaveStatus('Salvo ✓');
  }catch(e){
    console.error('Erro ao salvar no Supabase:', e);
    setSaveStatus('Erro ao salvar', true);
  }
}

async function loadStateFromSupabase(){
  if (!supabaseClient) return;
  try{
    setSaveStatus('Carregando…');
    const { data, error } = await supabaseClient
      .from(SUPABASE_TABLE)
      .select('data')
      .eq('id', SUPABASE_ROW_ID)
      .maybeSingle();
    if (error) throw error;
    if (data && data.data){
      state = data.data;
      /* uid precisa continuar maior que qualquer id já usado, para novos
         dias não colidirem com dias carregados da nuvem. */
      const maxId = state.days.reduce((m,d)=>Math.max(m, d.id||0), 0);
      uid = maxId + 1;
    }
    setSaveStatus('Sincronizado ✓');
  }catch(e){
    console.error('Erro ao carregar do Supabase:', e);
    setSaveStatus('Sem conexão com a nuvem (usando dados locais)', true);
  }
}

/* ======================= DATA MODEL ======================= */
let uid = 1;
function newTeam(name, hours){ return {name: name || '', hours: hours || [0,0,0]}; }
function newDay(name, period, teams){ return { id: uid++, name: name, period: period || '', teams: teams }; }

let state = {
  selected: 'overview',
  query: '',
  goals: { monthly: 60000, dailyAvg: 4000, turno: 10000 },
  days: [
    newDay('BASE', '00:00 - 03:00', [newTeam('DUPLA 1'), newTeam('DUPLA 2'), newTeam('DUPLA 3')]),
  ]
};

/* ======================= CALC ======================= */
function calcDay(day){
  const teams = day.teams;
  const hourTotals = [0,0,0];
  teams.forEach(t=>{ for(let i=0;i<3;i++){ hourTotals[i] += Number(t.hours[i])||0; } });
  const teamTotals = teams.map(t => t.hours.reduce((a,b)=>a+(Number(b)||0),0));
  const totalGeral = hourTotals.reduce((a,b)=>a+b,0);
  const mediaHora = totalGeral / 3;
  const trend1 = hourTotals[0] ? (hourTotals[1]-hourTotals[0])/hourTotals[0] : null;
  const trend2 = hourTotals[1] ? (hourTotals[2]-hourTotals[1])/hourTotals[1] : null;
  return {hourTotals, teamTotals, totalGeral, mediaHora, trend1, trend2};
}
function calcOverview(){
  const rows = state.days.map(d => ({day:d, calc: calcDay(d)}));
  const totalEtiquetas = rows.reduce((a,r)=>a+r.calc.totalGeral,0);
  const diasRegistrados = rows.length;
  const mediaPorDia = diasRegistrados ? totalEtiquetas/diasRegistrados : 0;
  const mediaHoraGeral = diasRegistrados ? rows.reduce((a,r)=>a+r.calc.mediaHora,0)/diasRegistrados : 0;
  let melhor = null;
  rows.forEach(r=>{ if(!melhor || r.calc.totalGeral > melhor.calc.totalGeral) melhor = r; });
  return {rows, totalEtiquetas, diasRegistrados, mediaPorDia, mediaHoraGeral, melhor};
}
function aggregateTeams(){
  const map = {};
  state.days.forEach(d=>{
    d.teams.forEach(t=>{
      const key = (t.name||'').trim().toUpperCase();
      if(!key) return;
      const total = t.hours.reduce((a,b)=>a+(Number(b)||0),0);
      map[key] = (map[key]||0) + total;
    });
  });
  return Object.entries(map)
    .map(([name,total])=>({name,total}))
    .filter(t=>t.total>0)
    .sort((a,b)=>b.total-a.total);
}

/* Name shown in the "Dupla destaque" card. Prefers the top producer (via
   aggregateTeams), but falls back to the first typed team name even with
   zero production yet, so a name never just fails to appear. */
function getHighlightTeamName(){
  const ranking = aggregateTeams();
  if(ranking.length) return ranking[0].name;
  for(const d of state.days){
    for(const t of d.teams){
      const nm = (t.name||'').trim();
      if(nm) return nm;
    }
  }
  return null;
}

/* ======================= FORMAT HELPERS ======================= */
const fmtInt = n => Math.round(n).toLocaleString('pt-BR');
const fmtPct = n => n===null || n===undefined ? '—' : (n*100).toFixed(1).replace('.',',') + '%';
const numLabel = v => Math.round(Number(v)||0).toLocaleString('pt-BR');
function escapeHtml(s){
  return String(s).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
}
function initials(name){
  const parts = String(name||'').trim().split(/\s+/).filter(Boolean);
  if(!parts.length) return 'ME';
  return (parts[0][0] + (parts[1] ? parts[1][0] : '')).toUpperCase();
}

/* ---- inline icon set (stroke-based, currentColor) ---- */
const ICONS = {
  search: '<path d="M11 19a8 8 0 1 1 0-16 8 8 0 0 1 0 16Z"/><path d="M21 21l-4.3-4.3"/>',
  bell: '<path d="M18 16v-5a6 6 0 1 0-12 0v5l-1.5 2.5h15L18 16Z"/><path d="M9.5 21a2.5 2.5 0 0 0 5 0"/>',
  refresh: '<path d="M20 11A8 8 0 0 0 6.3 6.3L4 8.6"/><path d="M4 4v4.6h4.6"/><path d="M4 13a8 8 0 0 0 13.7 4.7L20 15.4"/><path d="M20 20v-4.6h-4.6"/>',
  box: '<path d="M3.5 8 12 3.5 20.5 8 12 12.5 3.5 8Z"/><path d="M3.5 8v9L12 21.5 20.5 17V8"/><path d="M12 12.5V21.5"/>',
  bolt: '<path d="M13 2 4 14h6l-1 8 9-12h-6l1-8Z"/>',
  trophy: '<path d="M8 4h8v4a4 4 0 0 1-8 0V4Z"/><path d="M8 5H5a3 3 0 0 0 3 4"/><path d="M16 5h3a3 3 0 0 1-3 4"/><path d="M10 15h4v2h-4z"/><path d="M9 21h6"/><path d="M12 17v4"/>',
  users: '<circle cx="9" cy="8" r="3.2"/><path d="M3 20c0-3.3 2.7-6 6-6s6 2.7 6 6"/><circle cx="17.5" cy="9.5" r="2.5"/><path d="M15.5 14a5 5 0 0 1 5.5 5.5"/>',
  calendar: '<rect x="3.5" y="5" width="17" height="15.5" rx="2.5"/><path d="M3.5 10h17"/><path d="M8 3v4"/><path d="M16 3v4"/>',
  target: '<circle cx="12" cy="12" r="8.5"/><circle cx="12" cy="12" r="4.5"/><circle cx="12" cy="12" r=".8" fill="currentColor" stroke="none"/>',
  plus: '<path d="M12 5v14"/><path d="M5 12h14"/>',
  download: '<path d="M12 3v12.5"/><path d="M7.5 11 12 15.5 16.5 11"/><path d="M5 20h14"/>',
};
function icon(name, size){
  size = size || 16;
  return `<svg width="${size}" height="${size}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">${ICONS[name]||''}</svg>`;
}

/* ======================= RENDER: SIDEBAR ======================= */
function renderSidebar(){
  scheduleSave();
  document.getElementById('nav-overview').innerHTML = `
    <div class="nav-item ${state.selected==='overview'?'active':''}" onclick="selectView('overview')">
      <span class="ic">▦</span><span class="lbl">Visão Geral</span>
    </div>`;

  const q = state.query.trim().toLowerCase();
  const days = state.days.filter(d => !q || d.name.toLowerCase().includes(q));
  document.getElementById('nav-days').innerHTML = days.map(d=>{
    const c = calcDay(d);
    return `
    <div class="nav-item ${state.selected===d.id?'active':''}" onclick="selectView(${d.id})">
      <span class="ic">${c.totalGeral>0?'●':'○'}</span>
      <span class="lbl" title="${escapeHtml(d.name)}">${escapeHtml(d.name)}</span>
      <span class="rm" onclick="event.stopPropagation(); removeDay(${d.id})" title="Remover dia">×</span>
    </div>`;
  }).join('') || `<div style="color:rgba(255,255,255,.6); font-size:11.5px; padding:8px;">Nenhuma aba encontrada</div>`;
}

/* ======================= RENDER: RIGHT SIDEBAR ======================= */
function renderRightSidebar(){
  scheduleSave();
  const ov = calcOverview();
  const ranking = aggregateTeams().slice(0,4);
  const belowAvg = ov.rows.filter(r=>r.calc.totalGeral>0 && r.calc.totalGeral < ov.mediaPorDia).length;
  const pctMonthly = Math.min(100, (ov.totalEtiquetas / (state.goals.monthly||1)) * 100);
  const pctHourly = Math.min(100, (ov.mediaHoraGeral / (state.goals.dailyAvg||1)) * 100);
  const pctAboveAvg = ov.diasRegistrados ? Math.min(100, ((ov.diasRegistrados - belowAvg) / ov.diasRegistrados) * 100) : 0;
  const topName = getHighlightTeamName() || '—';

  document.getElementById('side-right').innerHTML = `
    <div class="sr-top">
      <div class="sr-icon" title="Dias abaixo da média">
        ${icon('bell',15)}
        ${belowAvg>0 ? `<span class="badge">${belowAvg}</span>` : ''}
      </div>
      <div class="sr-icon" title="Painel ao vivo">${icon('refresh',15)}</div>
    </div>

    <div class="profile-card">
      <div class="avatar-ring">${escapeHtml(initials(topName))}</div>
      <div class="pname">${topName!=='—' ? escapeHtml(topName) : 'Sem duplas ainda'}</div>
      <div class="prole">Dupla destaque</div>
      <div class="profile-trio">
        <div class="item"><b class="num">${ov.diasRegistrados}</b><span>Dias</span></div>
        <div class="item"><b class="num">${fmtInt(ov.totalEtiquetas)}</b><span>Total</span></div>
        <div class="item"><b class="num">${fmtInt(ov.mediaPorDia)}</b><span>Média</span></div>
      </div>
    </div>

    <div class="goals-block">
      <h5><span class="h5-ic">${icon('target',13)}</span>Metas de Produção</h5>
      <div class="goal-row">
        <div class="g-top">
          <span class="g-name">Meta mensal</span>
          <span class="g-val"><span class="num">${fmtInt(ov.totalEtiquetas)}</span> / <input class="g-target num" type="number" value="${state.goals.monthly}" oninput="setGoal('monthly',this.value)"></span>
        </div>
        <div class="goal-track"><div class="goal-fill" style="width:${pctMonthly}%"></div></div>
      </div>
      <div class="goal-row">
        <div class="g-top">
          <span class="g-name">Média/hora alvo</span>
          <span class="g-val"><span class="num">${fmtInt(ov.mediaHoraGeral)}</span> / <input class="g-target num" type="number" value="${state.goals.dailyAvg}" oninput="setGoal('dailyAvg',this.value)"></span>
        </div>
        <div class="goal-track"><div class="goal-fill" style="width:${pctHourly}%"></div></div>
      </div>
      <div class="goal-row">
        <div class="g-top">
          <span class="g-name">Dias acima da média</span>
          <span class="g-val">${ov.diasRegistrados - belowAvg} / ${ov.diasRegistrados}</span>
        </div>
        <div class="goal-track"><div class="goal-fill" style="width:${pctAboveAvg}%"></div></div>
      </div>
    </div>

    <div class="rank-block">
      <h5><span class="h5-ic">${icon('users',13)}</span>Ranking de Duplas</h5>
      ${ranking.length ? ranking.map((r,i)=>`
        <div class="rank-row">
          <div class="pos">${i+1}</div>
          <div class="rname" title="${escapeHtml(r.name)}">${escapeHtml(r.name)}</div>
          <div class="rval num">${fmtInt(r.total)}</div>
        </div>
      `).join('') : `<div class="rank-empty">Nenhuma dupla com produção lançada.</div>`}
    </div>
  `;
}

/* ======================= RENDER: MAIN ======================= */
function destroyAllCharts(){
  [chartBar, chartLine, chartAvg, chartWave, spark1, spark2, spark3].forEach(c=>{
    if(c){ try{ c.destroy(); }catch(e){ /* already gone, ignore */ } }
  });
  chartBar = chartLine = chartAvg = chartWave = spark1 = spark2 = spark3 = null;
}

function renderMain(){
  scheduleSave();
  const main = document.getElementById('main');
  /* Chart.js instances must be destroyed BEFORE their <canvas> is removed from
     the DOM (which innerHTML replacement below does) — otherwise a pending
     internal resize/animation callback can fire against a detached canvas and
     throw "this._fn is not a function". */
  destroyAllCharts();
  if(state.selected === 'overview'){
    main.innerHTML = renderOverviewHTML();
    drawCharts();
  } else {
    const day = state.days.find(d=>d.id===state.selected);
    if(!day){ state.selected='overview'; renderMain(); return; }
    main.innerHTML = renderDayHTML(day);
  }
  main.classList.remove('view-in');
  void main.offsetWidth; /* restart animation */
  main.classList.add('view-in');
  renderRightSidebar();
}

function greetingFor(date){
  const hour = date.getHours();
  if(hour < 5) return 'Boa madrugada';
  if(hour < 12) return 'Bom dia';
  if(hour < 18) return 'Boa tarde';
  return 'Boa noite';
}

function renderOverviewHTML(){
  const ov = calcOverview();
  const now = new Date();
  const saud = greetingFor(now);

  const last7 = ov.rows.slice(-7);
  const maxTotal = Math.max(1, ...last7.map(r=>r.calc.totalGeral));

  return `
    <div class="top-header">
      <div>
        <h2>Visão Geral</h2>
        <div class="datepill">${now.toLocaleDateString('pt-BR',{weekday:'long', day:'2-digit', month:'long'})}</div>
      </div>
      <div class="header-actions">
        <div class="search-wrap">
          <span class="search-ic">${icon('search',14)}</span>
          <input class="glass-input" placeholder="Buscar aba…" value="${escapeHtml(state.query)}" oninput="setQuery(this.value)">
        </div>
        <select class="glass-select" id="reportScope" title="Período do relatório">
          <option value="all">Mês completo</option>
          <option value="week">Última semana</option>
        </select>
        <button class="dl-btn" onclick="downloadOverviewPDF(document.getElementById('reportScope').value)">${icon('download',14)}Baixar Relatório</button>
        <div class="live-badge"><span class="live-dot"></span>Atualizado agora</div>
      </div>
    </div>

    <div class="hero">
      <div class="blob blob1"></div><div class="blob blob2"></div>
      <div class="hero-text">
        <h3 id="heroGreeting">${saud}, supervisor(a)!</h3>
        <p>${ov.diasRegistrados} abas ativas, ${fmtInt(ov.totalEtiquetas)} etiquetas registradas ao todo. Adicione um novo dia para continuar o acompanhamento.</p>
        <button class="hero-cta" onclick="addDay()">${icon('plus',14)}Adicionar novo dia</button>
      </div>
      <div class="hero-illustration">
        <svg width="150" height="140" viewBox="0 0 150 140" fill="none" xmlns="http://www.w3.org/2000/svg">
          <rect x="18" y="46" width="70" height="70" rx="10" fill="#38761D" opacity=".18"/>
          <rect x="40" y="30" width="70" height="70" rx="10" fill="#6AA84F"/>
          <path d="M58 50h34l14 14v22a4 4 0 0 1-4 4H58a4 4 0 0 1-4-4V54a4 4 0 0 1 4-4Z" fill="#fff" opacity=".92"/>
          <circle cx="66" cy="66" r="4.5" fill="#1F3864"/>
          <path d="M62 82h26M62 90h18" stroke="#1F3864" stroke-width="2.4" stroke-linecap="round"/>
          <path d="M92 50v14h14" stroke="#38761D" stroke-width="2.4" stroke-linejoin="round"/>
          <circle cx="120" cy="40" r="10" fill="#D9A521"/>
          <path d="M116 40l3 3 6-6" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </div>
    </div>

    <div class="row-2col">
      <div class="panel">
        <div class="panel-head">
          <h4><span class="h-ic">${icon('bolt',13)}</span>Atividade Recente</h4>
          <span class="tag">Últimos ${last7.length} dias</span>
        </div>
        <div class="capsules">
          ${last7.map(r=>{
            const pct = Math.max(4, (r.calc.totalGeral / maxTotal) * 100);
            const color = r.calc.totalGeral >= ov.mediaPorDia ? 'linear-gradient(180deg, var(--green-light), var(--green-dark))' : 'linear-gradient(180deg, var(--navy-soft), var(--navy))';
            const shortLbl = r.day.name.length>7 ? r.day.name.slice(0,6)+'…' : r.day.name;
            const tip = escapeHtml(r.day.name) + ': ' + fmtInt(r.calc.totalGeral) + ' etiquetas';
            return `
            <div class="capsule-col" title="${tip}">
              <div class="capsule-value">${fmtInt(r.calc.totalGeral)}</div>
              <div class="capsule-track"><div class="capsule-fill" style="height:${pct}%; background:${color};"></div></div>
              <div class="capsule-day">${escapeHtml(shortLbl)}</div>
            </div>`;
          }).join('')}
        </div>
        <div class="legend-row">
          <div class="legend-item"><span class="legend-dot" style="background:var(--green-dark);"></span>Acima da média</div>
          <div class="legend-item"><span class="legend-dot" style="background:var(--navy);"></span>Abaixo da média</div>
        </div>
      </div>

      <div class="stat-glass">
        <div class="deco deco1"></div>
        <div class="head"><span>ESTATÍSTICA DO TURNO</span><span class="pill">${ov.diasRegistrados} dias</span></div>
        <div class="big num">${fmtInt(ov.totalEtiquetas)}<small>etiquetas</small></div>
        <div class="chart-wrap"><canvas id="chartWave"></canvas></div>
        <div class="footnote"><span>${ov.rows[0] ? escapeHtml(ov.rows[0].day.name) : ''}</span><span>${ov.melhor ? escapeHtml(ov.melhor.day.name) : ''}</span></div>
      </div>
    </div>

    <div class="mini-row">
      <div class="mini-card c1">
        <div class="deco"></div>
        <div class="top"><span class="lbl"><span class="mini-ic">${icon('box',14)}</span>Produção Total</span><span class="delta">acumulado</span></div>
        <div class="val num">${fmtInt(ov.totalEtiquetas)}<small> und.</small></div>
        <div class="spark"><canvas id="spark1"></canvas></div>
      </div>
      <div class="mini-card c2">
        <div class="deco"></div>
        <div class="top"><span class="lbl"><span class="mini-ic">${icon('bolt',14)}</span>Média por Hora</span><span class="delta">geral</span></div>
        <div class="val num">${fmtInt(ov.mediaHoraGeral)}<small> und/h</small></div>
        <div class="spark"><canvas id="spark2"></canvas></div>
      </div>
      <div class="mini-card c3">
        <div class="deco"></div>
        <div class="top"><span class="lbl"><span class="mini-ic">${icon('trophy',14)}</span>Melhor Dia</span><span class="delta">${ov.melhor ? fmtInt(ov.melhor.calc.totalGeral) : 0}</span></div>
        <div class="val" style="font-size:15px;">${ov.melhor ? escapeHtml(ov.melhor.day.name) : '—'}</div>
        <div class="spark"><canvas id="spark3"></canvas></div>
      </div>
    </div>

    <div class="section-title">Análise Detalhada</div>
    <div class="chart-grid">
      <div class="card">
        <h3><span class="h-ic">${icon('target',13)}</span>Produção Total x Média do Dia</h3>
        <div class="chart-sub">Total por aba, comparado à média geral (${fmtInt(ov.mediaPorDia)})</div>
        <div class="chart-wrap"><canvas id="chartAvg"></canvas></div>
      </div>
      <div class="card">
        <h3><span class="h-ic">${icon('users',13)}</span>Produção por Dupla</h3>
        <div class="chart-sub">Contribuição de cada dupla, por dia</div>
        <div class="chart-wrap"><canvas id="chartBar"></canvas></div>
      </div>
      <div class="card">
        <h3><span class="h-ic">${icon('bolt',13)}</span>Tendência da Média por Hora</h3>
        <div class="chart-sub">Evolução da média/hora entre os dias</div>
        <div class="chart-wrap"><canvas id="chartLine"></canvas></div>
      </div>
    </div>

    <div class="section-title">Detalhamento por Dia</div>
    <div class="card table-card">
      <div class="table-scroll">
      <table>
        <thead>
          <tr>
            <th style="width:36px;">Nº</th><th style="text-align:left;">Aba</th><th>Período</th><th>Total Geral</th>
            <th>Média/Hora</th><th>Tend. 1H→2H</th><th>Tend. 2H→3H</th>
          </tr>
        </thead>
        <tbody>
          ${ov.rows.map((r,i)=>`
            <tr>
              <td>${i+1}</td>
              <td class="abacell" onclick="selectView(${r.day.id})">${escapeHtml(r.day.name)}</td>
              <td>${r.day.period ? escapeHtml(r.day.period) : '<span class="badge-empty">—</span>'}</td>
              <td><b class="num">${fmtInt(r.calc.totalGeral)}</b></td>
              <td class="num">${fmtInt(r.calc.mediaHora)}</td>
              <td class="num ${r.calc.trend1>0?'trend-up':(r.calc.trend1<0?'trend-down':'')}">${fmtPct(r.calc.trend1)}</td>
              <td class="num ${r.calc.trend2>0?'trend-up':(r.calc.trend2<0?'trend-down':'')}">${fmtPct(r.calc.trend2)}</td>
            </tr>
          `).join('')}
        </tbody>
        <tfoot>
          <tr><td colspan="3">TOTAL GERAL</td><td class="num">${fmtInt(ov.totalEtiquetas)}</td><td colspan="3"></td></tr>
        </tfoot>
      </table>
      </div>
    </div>
  `;
}

function renderDayHTML(day){
  const c = calcDay(day);
  const hourLabels = ['1ª HORA','2ª HORA','3ª HORA'];
  return `
    <div class="top-header">
      <div>
        <h2>Editar Dia</h2>
        <div class="datepill">Os totais são recalculados automaticamente</div>
      </div>
      <div class="header-actions">
        <button class="dl-btn" onclick="downloadDayPDF(${day.id})">${icon('download',14)}Relatório Diário</button>
        <div class="live-badge"><span class="live-dot"></span>Editando</div>
      </div>
    </div>
    <div class="day-header">
      <div class="deco deco1"></div>
      <div class="name-block">
        <span class="eyebrow">Nome da aba</span>
        <input class="title-input" value="${escapeHtml(day.name)}" oninput="renameDay(${day.id}, this.value)" placeholder="Nome da aba (ex: 30/07)">
      </div>
      <div class="period-field">
        Período total
        <input value="${escapeHtml(day.period)}" oninput="setPeriod(${day.id}, this.value)" placeholder="00:00 - 03:00">
      </div>
    </div>
    <div class="day-body">
      <table class="hours-table">
        <thead>
          <tr>
            <th class="timecol">Intervalo de Tempo</th>
            ${day.teams.map((t,ti)=>`<th><input class="teamname" value="${escapeHtml(t.name)}" oninput="setTeamName(${day.id},${ti},this.value)" placeholder="Nome da dupla"></th>`).join('')}
            <th style="width:120px;">Total da Hora</th>
          </tr>
        </thead>
        <tbody>
          ${[0,1,2].map(hi => `
            <tr>
              <td class="timecell">${hourLabels[hi]}</td>
              ${day.teams.map((t,ti)=>`<td><input class="num" type="number" inputmode="numeric" value="${t.hours[hi]}" oninput="setHour(${day.id},${ti},${hi},this.value)" onkeydown="handleCellKey(event)" onfocus="this.select()"></td>`).join('')}
              <td class="hourtotal num">${fmtInt(c.hourTotals[hi])}</td>
            </tr>
          `).join('')}
        </tbody>
        <tfoot>
          <tr>
            <td>TOTAL POR DUPLA</td>
            ${c.teamTotals.map(v=>`<td class="num">${fmtInt(v)}</td>`).join('')}
            <td class="num">${fmtInt(c.totalGeral)}</td>
          </tr>
        </tfoot>
      </table>

      <div class="toolbar-row">
        <button class="pill-btn" ${day.teams.length>=3?'disabled':''} onclick="addTeam(${day.id})">+ Adicionar dupla</button>
        ${day.teams.length>1 ? `<button class="pill-btn danger" onclick="removeTeam(${day.id})">− Remover última dupla</button>` : ''}
      </div>

      <div class="section-title" style="margin-top:0;">Resumo Operacional</div>
      <div class="summary-grid" style="grid-template-columns:repeat(2,1fr);">
        <div class="sum-card"><div class="lbl">Média por Hora</div><div class="val num">${fmtInt(c.mediaHora)}</div></div>
        <div class="sum-card"><div class="lbl">Total Geral</div><div class="val num">${fmtInt(c.totalGeral)}</div></div>
      </div>

      <div class="section-title">Meta de Produtividade por Hora</div>
      <div class="goal-editor-row">
        Meta do turno (3h):
        <input type="number" class="goal-turno-input" value="${state.goals.turno}" oninput="setTurnoGoal(this.value)">
        etiquetas &nbsp;→&nbsp; meta por hora: <b id="turnoGoalDisplay" class="num">${fmtInt((state.goals.turno||0)/3)}</b>
      </div>
      <div class="hour-goals">
        ${[0,1,2].map(hi=>{
          const label = ['1ª HORA','2ª HORA','3ª HORA'][hi];
          const val = c.hourTotals[hi];
          const goal = (state.goals.turno||0) / 3;
          const pct = goal>0 ? (val/goal)*100 : 0;
          const state_ = goal<=0 ? 'nogoal' : (val>=goal ? 'hit' : (val>0 ? 'mid' : 'miss'));
          const pctLabel = goal<=0 ? 'Meta ainda não definida' : (val<=0 ? 'Sem dados lançados' : Math.round(pct)+'% da meta atingido');
          return `
          <div class="hour-goal-row" data-hi="${hi}">
            <div class="hg-top">
              <span class="hg-label">${label}</span>
              <span class="hg-val">${fmtInt(val)}<span class="hg-of"> / meta ${fmtInt(goal)}</span></span>
            </div>
            <div class="hg-track"><div class="hg-fill ${state_}" style="width:${Math.max(3,Math.min(100,pct))}%"></div></div>
            <div class="hg-pct ${state_}">${pctLabel}</div>
          </div>`;
        }).join('')}
      </div>
    </div>
  `;
}

/* ======================= CHARTS ======================= */
let chartBar=null, chartLine=null, chartAvg=null, chartWave=null, spark1=null, spark2=null, spark3=null;
const CHARTS_AVAILABLE = typeof Chart !== 'undefined';
let DATALABELS_READY = false;

if(CHARTS_AVAILABLE){
  Chart.defaults.font.family = "'Segoe UI', Arial, sans-serif";
  Chart.defaults.color = '#66728A';
  Chart.defaults.animation = { duration: 650, easing: 'easeOutQuart' };
  Chart.defaults.plugins.legend.labels.usePointStyle = true;
  Chart.defaults.plugins.legend.labels.boxWidth = 8;
  Chart.defaults.plugins.legend.labels.boxHeight = 8;
  Chart.defaults.plugins.datalabels = { display: false };
  Chart.defaults.plugins.tooltip = {
    backgroundColor:'#132345',
    titleColor:'#FFFFFF',
    bodyColor:'#DCE4F5',
    footerColor:'#F2C94C',
    titleFont:{size:11.5, weight:'700'},
    bodyFont:{size:11.5},
    footerFont:{size:10.5, weight:'700'},
    padding:11,
    cornerRadius:9,
    displayColors:true,
    boxPadding:5,
    usePointStyle:true,
  };

  /* Load the datalabels plugin only AFTER Chart.js is confirmed loaded,
     so a missing/blocked plugin CDN can never crash the page. */
  const dlScript = document.createElement('script');
  dlScript.src = 'https://cdnjs.cloudflare.com/ajax/libs/chartjs-plugin-datalabels/2.2.0/chartjs-plugin-datalabels.min.js';
  dlScript.crossOrigin = 'anonymous';
  dlScript.onload = function(){
    try{
      if(typeof ChartDataLabels !== 'undefined'){
        Chart.register(ChartDataLabels);
        DATALABELS_READY = true;
        if(state.selected === 'overview') drawCharts();
      }
    }catch(e){ /* labels are a nice-to-have, never fatal */ }
  };
  dlScript.onerror = function(){ /* silently continue without data labels */ };
  document.head.appendChild(dlScript);
}

/* Load jspdf-autotable only AFTER jsPDF itself is confirmed loaded. */
let AUTOTABLE_READY = false;
if(window.jspdf){
  const atScript = document.createElement('script');
  atScript.src = 'https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.25/jspdf.plugin.autotable.min.js';
  atScript.crossOrigin = 'anonymous';
  atScript.onload = function(){ AUTOTABLE_READY = true; };
  atScript.onerror = function(){ /* PDF export buttons will warn the user if this never becomes ready */ };
  document.head.appendChild(atScript);
}

function vGradient(ctx, chartArea, stops){
  if(!chartArea) return stops[stops.length-1][1];
  const g = ctx.createLinearGradient(0, chartArea.top, 0, chartArea.bottom);
  stops.forEach(([stop,color])=> g.addColorStop(stop,color));
  return g;
}

function chartFallback(canvasId, mini){
  const el = document.getElementById(canvasId);
  if(!el) return;
  const box = document.createElement('div');
  if(mini){
    box.className = 'spark-offline';
  } else {
    box.className = 'chart-offline';
    box.innerHTML = '<span class="chart-offline-ic">'+icon('bolt',13)+'</span> Gráfico indisponível — sem conexão com a biblioteca de gráficos.';
  }
  el.replaceWith(box);
}

function chartEmptyState(canvasId){
  const el = document.getElementById(canvasId);
  if(!el) return;
  const box = document.createElement('div');
  box.className = 'chart-offline';
  box.innerHTML = '<span class="chart-offline-ic">'+icon('calendar',13)+'</span> Adicione um dia para ver este gráfico.';
  el.replaceWith(box);
}

function sparkline(id, data, labels){
  const el = document.getElementById(id);
  if(!el) return null;
  return new Chart(el, {
    type:'line',
    data:{ labels: labels || data.map((_,i)=>i), datasets:[{
      data, borderColor:'rgba(255,255,255,.95)',
      backgroundColor:(c)=>{ const {ctx,chartArea}=c.chart; return vGradient(ctx,chartArea,[[0,'rgba(255,255,255,.35)'],[1,'rgba(255,255,255,0)']]); },
      borderWidth:2, pointRadius:0, pointHoverRadius:4, pointHoverBackgroundColor:'#fff',
      pointHitRadius:10, tension:.35, fill:true
    }]},
    options:{
      responsive:true, maintainAspectRatio:false,
      interaction:{mode:'index', intersect:false},
      plugins:{legend:{display:false}, tooltip:{
        backgroundColor:'rgba(15,30,60,.92)', displayColors:false,
        callbacks:{ title:(items)=>items[0].label||'', label:(item)=>' '+numLabel(item.raw) }
      }},
      scales:{ x:{display:false}, y:{display:false} },
      elements:{line:{borderCapStyle:'round'}}
    }
  });
}

function drawCharts(){
  const ov = calcOverview();
  const labels = ov.rows.map(r=>r.day.name);
  const palette = ['#1F3864','#6AA84F','#D9A521'];

  if(!CHARTS_AVAILABLE){
    ['chartAvg','chartBar','chartLine','chartWave'].forEach(id=>chartFallback(id,false));
    ['spark1','spark2','spark3'].forEach(id=>chartFallback(id,true));
    return;
  }

  if(ov.rows.length === 0){
    ['chartAvg','chartBar','chartLine','chartWave'].forEach(id=>chartEmptyState(id));
    ['spark1','spark2','spark3'].forEach(id=>chartFallback(id,true));
    return;
  }

  /* wavy stat card chart */
  const ctxWave = document.getElementById('chartWave');
  if(chartWave) chartWave.destroy();
  if(ctxWave){
    chartWave = new Chart(ctxWave, {
      type:'line',
      data:{ labels, datasets:[{
        data: ov.rows.map(r=>r.calc.totalGeral),
        borderColor:'rgba(255,255,255,.95)',
        backgroundColor:(c)=>{ const {ctx,chartArea}=c.chart; return vGradient(ctx,chartArea,[[0,'rgba(255,255,255,.32)'],[1,'rgba(255,255,255,0)']]); },
        borderWidth:2.4, pointRadius:0, pointHoverRadius:5, pointHoverBackgroundColor:'#F2C94C',
        pointHoverBorderColor:'#fff', pointHitRadius:14, tension:.42, fill:true
      }]},
      options:{
        responsive:true, maintainAspectRatio:false,
        interaction:{mode:'index', intersect:false},
        plugins:{
          legend:{display:false},
          tooltip:{ backgroundColor:'rgba(15,30,60,.92)', callbacks:{
            title:(items)=>items[0].label,
            label:(item)=>' '+numLabel(item.raw)+' etiquetas'
          }}
        },
        scales:{ x:{display:false}, y:{display:false} }
      }
    });
  }

  if(spark1) spark1.destroy();
  if(spark2) spark2.destroy();
  if(spark3) spark3.destroy();
  spark1 = sparkline('spark1', ov.rows.map(r=>r.calc.totalGeral), labels);
  spark2 = sparkline('spark2', ov.rows.map(r=>r.calc.mediaHora), labels);
  const topTeam = aggregateTeams()[0];
  const topSeries = ov.rows.map(r=>{
    const t = r.day.teams.find(t=> (t.name||'').trim().toUpperCase() === (topTeam? topTeam.name : ''));
    return t ? t.hours.reduce((a,b)=>a+(Number(b)||0),0) : 0;
  });
  spark3 = sparkline('spark3', topTeam ? topSeries : [0,0,0], labels);

  /* total vs average */
  const ctxAvg = document.getElementById('chartAvg');
  if(chartAvg) chartAvg.destroy();
  chartAvg = new Chart(ctxAvg, {
    data: { labels, datasets: [
      { type:'bar', label:'Total do dia', data: ov.rows.map(r=>r.calc.totalGeral),
        backgroundColor:(c)=>{
          const idx=c.dataIndex; if(idx===undefined) return '#6AA84F';
          const above = ov.rows[idx].calc.totalGeral >= ov.mediaPorDia;
          const {ctx,chartArea}=c.chart;
          return vGradient(ctx,chartArea, above ? [[0,'#8FCB6C'],[1,'#38761D']] : [[0,'#CBD5E5'],[1,'#93A3BE']]);
        },
        hoverBackgroundColor: (c)=>{ const idx=c.dataIndex; const above = ov.rows[idx].calc.totalGeral >= ov.mediaPorDia; return above ? '#57902E' : '#7C8CAA'; },
        borderRadius:5, borderSkipped:false, maxBarThickness:34, order:2,
        datalabels:{ display:true, anchor:'end', align:'top', color:'#1F3864', font:{size:9,weight:'700'}, formatter:v=>numLabel(v) }
      },
      { type:'line', label:'Média geral', data: ov.rows.map(()=>ov.mediaPorDia),
        borderColor:'#D9A521', borderWidth:2.5, borderDash:[6,4], pointRadius:0, tension:0, order:1,
        datalabels:{display:false} }
    ]},
    options:{ responsive:true, maintainAspectRatio:false,
      interaction:{mode:'index', intersect:false},
      plugins:{
        legend:{position:'bottom', labels:{font:{size:10.5}}},
        tooltip:{ callbacks:{
          label:(item)=> item.dataset.label==='Média geral' ? ' Média geral: '+numLabel(item.raw) : ' Total: '+numLabel(item.raw),
          footer:(items)=>{ const bar=items.find(i=>i.dataset.label==='Total do dia'); if(!bar) return '';
            const diff = bar.raw - ov.mediaPorDia; const pct = ov.mediaPorDia ? (diff/ov.mediaPorDia*100) : 0;
            return (diff>=0?'▲ ':'▼ ')+Math.abs(pct).toFixed(1).replace('.',',')+'% vs média'; }
        }}
      },
      scales:{
        x:{grid:{display:false}, ticks:{font:{size:9.5}, maxRotation:45, minRotation:30}},
        y:{grid:{color:'#EEF0F3', drawTicks:false, borderDash:[3,4]}, ticks:{font:{size:10}, callback:v=>numLabel(v)}, beginAtZero:true}
      }
    }
  });

  /* production by team stacked */
  const maxTeams = Math.max(1, ...ov.rows.map(r=>r.day.teams.length));
  const teamGradients = [
    [[0,'#3E5C95'],[1,'#1F3864']],
    [[0,'#8FCB6C'],[1,'#38761D']],
    [[0,'#F2C94C'],[1,'#B8860F']],
  ];
  const datasets = [];
  for(let ti=0; ti<maxTeams; ti++){
    datasets.push({
      label:'Dupla '+(ti+1), data: ov.rows.map(r=>r.calc.teamTotals[ti]||0),
      backgroundColor:(c)=>{ const {ctx,chartArea}=c.chart; return vGradient(ctx,chartArea, teamGradients[ti%teamGradients.length]); },
      hoverBackgroundColor: palette[ti%palette.length],
      stack:'a', borderRadius:3, maxBarThickness:30
    });
  }
  const ctxBar = document.getElementById('chartBar');
  if(chartBar) chartBar.destroy();
  chartBar = new Chart(ctxBar, {
    type:'bar', data:{ labels, datasets },
    options:{ responsive:true, maintainAspectRatio:false,
      interaction:{mode:'index', intersect:false},
      plugins:{
        legend:{position:'bottom', labels:{font:{size:10.5}}},
        tooltip:{ callbacks:{
          label:(item)=> ' '+item.dataset.label+': '+numLabel(item.raw),
          footer:(items)=>' Total: '+numLabel(items.reduce((a,i)=>a+i.raw,0))
        }}
      },
      scales:{
        x:{stacked:true, grid:{display:false}, ticks:{font:{size:9.5}, maxRotation:45, minRotation:30}},
        y:{stacked:true, grid:{color:'#EEF0F3', drawTicks:false, borderDash:[3,4]}, ticks:{font:{size:10}, callback:v=>numLabel(v)}}
      }
    }
  });

  /* hourly average trend */
  const ctxLine = document.getElementById('chartLine');
  if(chartLine) chartLine.destroy();
  chartLine = new Chart(ctxLine, {
    type:'line',
    data:{ labels, datasets:[{ label:'Média / Hora', data: ov.rows.map(r=>r.calc.mediaHora),
      borderColor:'#1F3864',
      backgroundColor:(c)=>{ const {ctx,chartArea}=c.chart; return vGradient(ctx,chartArea,[[0,'rgba(31,56,100,.20)'],[1,'rgba(31,56,100,0)']]); },
      pointBackgroundColor:'#D9A521', pointBorderColor:'#1F3864', pointBorderWidth:2,
      pointRadius:4.5, pointHoverRadius:7, pointHitRadius:12, borderWidth:2.5, tension:.3, fill:true,
      datalabels:{ display:true, align:'top', offset:6, color:'#1F3864', font:{size:8.5,weight:'700'}, formatter:v=>numLabel(v) }
    }]},
    options:{ responsive:true, maintainAspectRatio:false,
      interaction:{mode:'index', intersect:false},
      plugins:{
        legend:{display:false},
        tooltip:{ callbacks:{ label:(item)=>' Média/hora: '+numLabel(item.raw) } }
      },
      layout:{padding:{top:14}},
      scales:{
        x:{grid:{display:false}, ticks:{font:{size:9.5}, maxRotation:45, minRotation:30}},
        y:{grid:{color:'#EEF0F3', drawTicks:false, borderDash:[3,4]}, ticks:{font:{size:10}, callback:v=>numLabel(v)}, beginAtZero:true}
      }
    }
  });
}

/* ======================= ACTIONS ======================= */
function selectView(id){ state.selected = id; render(); }
function setQuery(v){ state.query = v; renderSidebar(); }
function setGoal(key, val){ state.goals[key] = Number(val)||0; renderRightSidebar(); }

function addDay(){
  const n = state.days.length + 1;
  const d = newDay('Novo dia ' + n, '', [newTeam('DUPLA 1'), newTeam('DUPLA 2')]);
  state.days.push(d);
  state.selected = d.id;
  render();
}
function removeDay(id){
  const day = state.days.find(d=>d.id===id);
  if(!day) return;
  if(!confirm('Remover a aba "' + day.name + '"? Esta ação não pode ser desfeita.')) return;
  state.days = state.days.filter(d=>d.id!==id);
  if(state.selected===id) state.selected='overview';
  render();
}
function renameDay(id, val){ const day = state.days.find(d=>d.id===id); if(day) day.name = val; renderSidebar(); }
function setPeriod(id, val){ const day = state.days.find(d=>d.id===id); if(day) day.period = val; scheduleSave(); }
function setTeamName(id, ti, val){
  const day = state.days.find(d=>d.id===id);
  if(day) day.teams[ti].name = val;
  renderRightSidebar();
}
function setTurnoGoal(val){
  state.goals.turno = val === '' ? 0 : Number(val);
  const day = state.days.find(d=>d.id===state.selected);
  if(day) patchDayComputed(day);
}

function setHour(id, ti, hi, val){
  const day = state.days.find(d=>d.id===id);
  if(!day) return;
  day.teams[ti].hours[hi] = val === '' ? 0 : Number(val);
  patchDayComputed(day);
  renderSidebar();
}

/* Updates only the computed (read-only) numbers in the day view — never touches
   the input elements themselves, so typing never loses focus/cursor position. */
function patchDayComputed(day){
  const c = calcDay(day);
  const hourEls = document.querySelectorAll('.hourtotal');
  hourEls.forEach((el,i)=>{ if(c.hourTotals[i] !== undefined) el.textContent = fmtInt(c.hourTotals[i]); });

  const footCells = document.querySelectorAll('.hours-table tfoot td');
  c.teamTotals.forEach((v,i)=>{ if(footCells[i+1]) footCells[i+1].textContent = fmtInt(v); });
  if(footCells.length) footCells[footCells.length-1].textContent = fmtInt(c.totalGeral);

  const sumCards = document.querySelectorAll('.summary-grid .sum-card');
  const sumVals = [fmtInt(c.mediaHora), fmtInt(c.totalGeral)];
  sumCards.forEach((card,i)=>{
    const v = card.querySelector('.val');
    if(v && sumVals[i] !== undefined) v.textContent = sumVals[i];
  });

  const goal = (state.goals.turno||0) / 3;
  const goalDisplay = document.getElementById('turnoGoalDisplay');
  if(goalDisplay) goalDisplay.textContent = fmtInt(goal);
  document.querySelectorAll('.hour-goal-row').forEach((row)=>{
    const hi = Number(row.getAttribute('data-hi'));
    const val = c.hourTotals[hi];
    if(val === undefined) return;
    const pct = goal>0 ? (val/goal)*100 : 0;
    const st = goal<=0 ? 'nogoal' : (val>=goal ? 'hit' : (val>0 ? 'mid' : 'miss'));
    const pctLabel = goal<=0 ? 'Meta ainda não definida' : (val<=0 ? 'Sem dados lançados' : Math.round(pct)+'% da meta atingido');

    const valEl = row.querySelector('.hg-val');
    if(valEl) valEl.innerHTML = fmtInt(val) + '<span class="hg-of"> / meta ' + fmtInt(goal) + '</span>';
    const fillEl = row.querySelector('.hg-fill');
    if(fillEl){
      fillEl.className = 'hg-fill ' + st;
      fillEl.style.width = Math.max(3, Math.min(100, pct)) + '%';
    }
    const pctEl = row.querySelector('.hg-pct');
    if(pctEl){ pctEl.className = 'hg-pct ' + st; pctEl.textContent = pctLabel; }
  });
}

/* Lets Enter (and Shift+Enter to go back) move between hour cells like a
   spreadsheet, so a whole table can be filled without touching the mouse. */
function handleCellKey(e){
  if(e.key !== 'Enter') return;
  e.preventDefault();
  const inputs = Array.from(document.querySelectorAll('.hours-table input.num'));
  const idx = inputs.indexOf(e.target);
  if(idx === -1) return;
  const nextIdx = e.shiftKey ? idx - 1 : idx + 1;
  if(nextIdx >= 0 && nextIdx < inputs.length) inputs[nextIdx].focus();
}
function addTeam(id){
  const day = state.days.find(d=>d.id===id);
  if(!day || day.teams.length>=3) return;
  day.teams.push(newTeam('DUPLA ' + (day.teams.length+1)));
  render();
}
function removeTeam(id){
  const day = state.days.find(d=>d.id===id);
  if(!day || day.teams.length<=1) return;
  day.teams.pop();
  render();
}

/* ======================= PDF EXPORT ======================= */
const PDF = {
  navy:[31,56,100], navyDark:[15,30,61], navySoft:[46,76,130],
  green:[106,168,79], greenDark:[56,118,29], gold:[217,165,33],
  lightGreen:[229,241,224], grayBg:[248,249,248], border:[224,228,233],
  text:[27,33,48], mute:[104,114,133], danger:[192,57,43],
};

function pdfLogoDataUrl(){
  const el = document.querySelector('.logo-badge');
  return (el && el.src && el.src.indexOf('data:image') === 0) ? el.src : null;
}
function pdfDrawFallbackBadge(doc, x, y, size){
  doc.setFillColor(...PDF.green);
  doc.roundedRect(x, y, size, size, size*0.24, size*0.24, 'F');
  doc.setTextColor(255,255,255); doc.setFont('helvetica','bold'); doc.setFontSize(size*0.37);
  doc.text('ME', x+size/2, y+size/2+size*0.13, {align:'center'});
}
function pdfDrawLogo(doc, x, y, size){
  const dataUrl = pdfLogoDataUrl();
  if(dataUrl){
    try{ doc.addImage(dataUrl, 'PNG', x, y, size, size); return; }catch(e){ /* fall through to badge */ }
  }
  pdfDrawFallbackBadge(doc, x, y, size);
}

function pdfHeader(doc, title, subtitle, docCode){
  const pw = doc.internal.pageSize.getWidth();
  doc.setFillColor(...PDF.navy); doc.rect(0,0,pw,92,'F');
  doc.setFillColor(...PDF.navyDark); doc.rect(0,88,pw,4,'F');
  doc.setFillColor(...PDF.gold); doc.rect(0,92,pw,2.6,'F');

  pdfDrawLogo(doc, 33, 18, 56);

  const textX = 100;
  doc.setFont('helvetica','bold'); doc.setFontSize(16); doc.setTextColor(255,255,255);
  doc.text('MESA DE ETIQUETAGEM', textX, 41);
  doc.setFont('helvetica','normal'); doc.setFontSize(10.5); doc.setTextColor(214,223,240);
  doc.text(title, textX, 57);
  doc.setFontSize(8.5); doc.setTextColor(172,187,217);
  doc.text(subtitle, textX, 71);

  if(docCode){
    doc.setFont('helvetica','normal'); doc.setFontSize(7.8); doc.setTextColor(172,187,217);
    doc.text(docCode, pw-40, 38, {align:'right'});
    doc.text(new Date().toLocaleDateString('pt-BR'), pw-40, 50, {align:'right'});
  }
  return 118;
}

function pdfFooter(doc){
  const pages = doc.internal.getNumberOfPages();
  const w = doc.internal.pageSize.getWidth();
  const h = doc.internal.pageSize.getHeight();
  for(let i=1;i<=pages;i++){
    doc.setPage(i);
    doc.setDrawColor(...PDF.border); doc.setLineWidth(0.6);
    doc.line(40, h-38, w-40, h-38);
    doc.setFont('helvetica','normal'); doc.setFontSize(7.8); doc.setTextColor(...PDF.mute);
    doc.text('Painel de Controle — Mesa de Etiquetagem', 40, h-24);
    doc.text('Documento gerado automaticamente', w/2, h-24, {align:'center'});
    doc.text('Página ' + i + ' de ' + pages, w-40, h-24, {align:'right'});
  }
}

function pdfSectionTitle(doc, x, y, text){
  const pw = doc.internal.pageSize.getWidth();
  doc.setFillColor(...PDF.greenDark);
  doc.rect(x, y-10, 3, 13, 'F');
  doc.setFont('helvetica','bold'); doc.setFontSize(10.5); doc.setTextColor(...PDF.navy);
  doc.text(text.toUpperCase(), x+9, y);
  doc.setDrawColor(...PDF.border); doc.setLineWidth(0.6);
  doc.line(x, y+6, pw-40, y+6);
  return y+24;
}

function pdfFit(doc, text, maxWidth){
  text = String(text);
  if(doc.getTextWidth(text) <= maxWidth) return text;
  let t = text;
  while(t.length>1 && doc.getTextWidth(t+'…') > maxWidth){ t = t.slice(0,-1); }
  return t + '…';
}

function pdfContinuationBand(doc, pw, label){
  doc.setFillColor(...PDF.navy); doc.rect(0,0,pw,46,'F');
  doc.setFillColor(...PDF.gold); doc.rect(0,46,pw,2,'F');
  pdfDrawLogo(doc, 24, 8, 30);
  doc.setFont('helvetica','bold'); doc.setFontSize(10.5); doc.setTextColor(255,255,255);
  doc.text('MESA DE ETIQUETAGEM', 62, 27);
  doc.setFont('helvetica','normal'); doc.setFontSize(8.5); doc.setTextColor(190,203,228);
  doc.text(label + ' (continuação)', pw-40, 27, {align:'right'});
  return 68;
}

function pdfKpiCards(doc, x, y, totalWidth, cards, h){
  h = h || 56;
  const gap = 10;
  const w = (totalWidth - gap*(cards.length-1)) / cards.length;
  cards.forEach((card,i)=>{
    const cx = x + i*(w+gap);
    doc.setFillColor(...PDF.grayBg); doc.setDrawColor(...PDF.border); doc.setLineWidth(0.6);
    doc.roundedRect(cx, y, w, h, 6, 6, 'FD');
    doc.setFillColor(...(card.accent||PDF.green));
    doc.roundedRect(cx, y, 4, h, 2, 2, 'F');
    doc.setFont('helvetica','bold'); doc.setFontSize(7.3); doc.setTextColor(...PDF.mute);
    doc.text(pdfFit(doc, card.label.toUpperCase(), w-20), cx+13, y+17);
    doc.setFont('helvetica','bold'); doc.setFontSize(15.5); doc.setTextColor(...PDF.navy);
    doc.text(pdfFit(doc, card.value, w-20), cx+13, y+38);
  });
  return y + h;
}

/* Draws one progress bar per hour comparing actual production to the
   fixed hourly goal (turno total ÷ 3) — mirrors the app's "Meta de
   Produtividade por Hora" cards. Returns the y position after the bars. */
function pdfHourGoalBars(doc, x, y, width, hourTotals){
  const goal = (state.goals.turno || 0) / 3;
  const labels = ['1ª HORA','2ª HORA','3ª HORA'];
  const labelW = 76, valueW = 118;
  const barX = x + labelW, barW = width - labelW - valueW;
  hourTotals.forEach((val,i)=>{
    const ry = y + i*27 + 9;
    doc.setFont('helvetica','bold'); doc.setFontSize(9.2); doc.setTextColor(...PDF.navy);
    doc.text(labels[i], x, ry);

    doc.setFillColor(...PDF.grayBg); doc.setDrawColor(...PDF.border); doc.setLineWidth(0.6);
    doc.roundedRect(barX, ry-9, barW, 13, 4, 4, 'FD');
    const color = goal<=0 ? PDF.mute : (val>=goal ? PDF.greenDark : (val>0 ? PDF.gold : [217,224,218]));
    const fillW = goal>0 ? Math.max(5, Math.min(barW, (val/goal)*barW)) : 4;
    doc.setFillColor(...color);
    doc.roundedRect(barX, ry-9, fillW, 13, 4, 4, 'F');

    doc.setFont('helvetica','bold'); doc.setFontSize(8.4); doc.setTextColor(...PDF.navy);
    const label = goal>0 ? (numLabel(val) + ' / ' + numLabel(goal)) : (numLabel(val) + ' / —');
    doc.text(label, x+width, ry, {align:'right'});
  });
  return y + hourTotals.length*27 + 8;
}

function downloadDayPDF(id){
  const day = state.days.find(d=>d.id===id);
  if(!day) return;
  if(!window.jspdf){ alert('Não foi possível carregar a biblioteca de geração de PDF (sem conexão com a internet). Verifique sua conexão e tente novamente.'); return; }
  if(!AUTOTABLE_READY){ alert('A biblioteca de tabelas do PDF ainda está carregando. Aguarde um instante e tente novamente.'); return; }
  const c = calcDay(day);
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF({unit:'pt', format:'a4'});
  const pw = doc.internal.pageSize.getWidth();
  const docCode = 'REL-DIA-' + day.name.replace(/[^A-Za-z0-9]+/g,'').slice(0,12).toUpperCase();

  let y = pdfHeader(doc, 'Relatório Diário de Produção', 'Relatório individual por aba / dia de operação', docCode);

  doc.setFont('helvetica','bold'); doc.setFontSize(19); doc.setTextColor(...PDF.text);
  doc.text(pdfFit(doc, day.name, pw-80), 40, y);
  y += 10;
  doc.setDrawColor(...PDF.border); doc.setLineWidth(0.6);
  doc.line(40, y, pw-40, y);
  y += 20;

  doc.setFont('helvetica','normal'); doc.setFontSize(9.3); doc.setTextColor(...PDF.mute);
  doc.text('PERÍODO TOTAL', 40, y);
  doc.text('DUPLAS ATIVAS', 230, y);
  doc.text('GERADO EM', 400, y);
  y += 14;
  doc.setFont('helvetica','bold'); doc.setFontSize(10.5); doc.setTextColor(...PDF.text);
  doc.text(day.period || '—', 40, y);
  doc.text(String(day.teams.length), 230, y);
  doc.text(new Date().toLocaleString('pt-BR'), 400, y);
  y += 30;

  y = pdfSectionTitle(doc, 40, y, 'Detalhamento por Hora');

  const head = [['Intervalo de Tempo', ...day.teams.map(t=>t.name||'Dupla'), 'Total da Hora']];
  const body = [0,1,2].map(hi=>{
    const label = ['1ª HORA','2ª HORA','3ª HORA'][hi];
    return [label, ...day.teams.map(t=>numLabel(t.hours[hi]||0)), numLabel(c.hourTotals[hi])];
  });
  const totalRow = ['TOTAL POR DUPLA', ...c.teamTotals.map(v=>numLabel(v)), numLabel(c.totalGeral)];

  doc.autoTable({
    startY:y, head, body:[...body, totalRow],
    theme:'grid', margin:{left:40, right:40, top:64},
    styles:{font:'helvetica', fontSize:10, cellPadding:8, lineColor:PDF.border, lineWidth:0.6, textColor:PDF.text, valign:'middle'},
    headStyles:{fillColor:PDF.navy, textColor:255, fontStyle:'bold', fontSize:9.5, halign:'center'},
    bodyStyles:{halign:'center'},
    columnStyles:{0:{halign:'left', fontStyle:'bold', textColor:PDF.navy}},
    alternateRowStyles:{fillColor:PDF.grayBg},
    didParseCell:(data)=>{
      if(data.section==='body' && data.row.index===body.length){
        data.cell.styles.fillColor=PDF.greenDark; data.cell.styles.textColor=255; data.cell.styles.fontStyle='bold';
      }
    },
    didDrawPage:(data)=>{ if(data.pageNumber>1) pdfContinuationBand(doc, pw, 'Relatório Diário de Produção'); }
  });

  y = doc.lastAutoTable.finalY + 30;
  if(y > doc.internal.pageSize.getHeight() - 220){ doc.addPage(); y = 50; }

  const turnoGoal = state.goals.turno || 0;
  const hourlyGoal = turnoGoal / 3;
  y = pdfSectionTitle(doc, 40, y, 'Meta de Produtividade por Hora');
  doc.setFont('helvetica','normal'); doc.setFontSize(8.8); doc.setTextColor(...PDF.mute);
  doc.text('Meta do turno (3h): ' + numLabel(turnoGoal) + ' etiquetas  →  meta por hora: ' + (hourlyGoal>0 ? numLabel(hourlyGoal) : '—'), 40, y);
  y += 18;
  y = pdfHourGoalBars(doc, 40, y, pw-80, c.hourTotals);

  pdfFooter(doc);
  doc.save('Relatorio_Diario_' + day.name.replace(/[^\w\-]+/g,'_') + '.pdf');
}

function downloadOverviewPDF(scope){
  if(!window.jspdf){ alert('Não foi possível carregar a biblioteca de geração de PDF (sem conexão com a internet). Verifique sua conexão e tente novamente.'); return; }
  if(!AUTOTABLE_READY){ alert('A biblioteca de tabelas do PDF ainda está carregando. Aguarde um instante e tente novamente.'); return; }
  const ov = calcOverview();
  const rows = scope==='week' ? ov.rows.slice(-7) : ov.rows;
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF({unit:'pt', format:'a4'});
  const pw = doc.internal.pageSize.getWidth();
  const scopeLabel = scope==='week' ? 'Semanal' : 'Mensal';
  const docCode = 'REL-' + scopeLabel.toUpperCase().slice(0,3) + '-' + new Date().toISOString().slice(0,10).replace(/-/g,'');

  let y = pdfHeader(doc, 'Relatório ' + scopeLabel + ' Consolidado',
    rows.length + ' aba(s) incluída(s) neste relatório', docCode);

  const totalSel = rows.reduce((a,r)=>a+r.calc.totalGeral,0);
  const mediaSel = rows.length ? totalSel/rows.length : 0;
  let melhorSel = null;
  rows.forEach(r=>{ if(!melhorSel || r.calc.totalGeral > melhorSel.calc.totalGeral) melhorSel = r; });

  const turnoGoal = state.goals.turno || 0;
  const hourlyGoal = turnoGoal / 3;
  const hitsPerRow = rows.map(r => hourlyGoal>0 ? r.calc.hourTotals.filter(v=>v>=hourlyGoal).length : 0);
  const totalHoursEval = rows.length * 3;
  const hoursHit = hitsPerRow.reduce((a,b)=>a+b,0);
  const adherencePct = totalHoursEval ? (hoursHit/totalHoursEval*100) : 0;

  pdfKpiCards(doc, 40, y, pw-80, [
    { label:'Total de Etiquetas', value: numLabel(totalSel), accent: PDF.navy },
    { label:'Dias no Relatório', value: String(rows.length), accent: PDF.greenDark },
    { label:'Média por Dia', value: numLabel(mediaSel), accent: PDF.gold },
    { label:'Melhor Dia', value: melhorSel ? melhorSel.day.name : '—', accent: PDF.greenDark },
  ], 58);
  y += 58 + 14;

  pdfKpiCards(doc, 40, y, pw-80, [
    { label:'Meta por Hora (Turno)', value: hourlyGoal>0 ? numLabel(hourlyGoal) : '—', accent: PDF.gold },
    { label:'Aderência à Meta', value: hourlyGoal>0 ? (Math.round(adherencePct)+'% ('+hoursHit+'/'+totalHoursEval+'h)') : 'Meta não definida', accent: PDF.greenDark },
  ], 46);
  y += 46 + 28;

  y = pdfSectionTitle(doc, 40, y, 'Detalhamento por Dia');

  const head = [['Nº','Aba','Período','Total Geral','Média/Hora','Horas na Meta']];
  const body = rows.map((r,i)=>[i+1, r.day.name, r.day.period||'—', numLabel(r.calc.totalGeral), numLabel(r.calc.mediaHora), hourlyGoal>0 ? (hitsPerRow[i]+'/3') : '—']);
  doc.autoTable({
    startY:y, head, body, theme:'grid', margin:{left:40, right:40, top:64},
    styles:{font:'helvetica', fontSize:9, cellPadding:6.5, lineColor:PDF.border, lineWidth:0.6, textColor:PDF.text, valign:'middle'},
    headStyles:{fillColor:PDF.navy, textColor:255, fontSize:8.8, fontStyle:'bold', halign:'center'},
    bodyStyles:{halign:'center'},
    columnStyles:{0:{cellWidth:24}, 1:{halign:'left', fontStyle:'bold', textColor:PDF.navy}},
    alternateRowStyles:{fillColor:PDF.grayBg},
    didParseCell:(data)=>{
      if(data.section==='body' && data.column.index===5 && hourlyGoal>0){
        const hits = hitsPerRow[data.row.index];
        data.cell.styles.fontStyle = 'bold';
        data.cell.styles.textColor = hits>=3 ? PDF.greenDark : (hits>0 ? [156,122,14] : PDF.mute);
      }
    },
    foot:[['','','TOTAL GERAL', numLabel(totalSel), '', hourlyGoal>0 ? (hoursHit+'/'+totalHoursEval) : '—']],
    footStyles:{fillColor:[241,243,240], textColor:PDF.navy, fontStyle:'bold', fontSize:9, halign:'center'},
    didDrawPage:(data)=>{ if(data.pageNumber>1) pdfContinuationBand(doc, pw, 'Relatório ' + scopeLabel + ' Consolidado'); }
  });

  const ranking = aggregateTeams().slice(0,5);
  if(ranking.length){
    y = doc.lastAutoTable.finalY + 30;
    if(y > doc.internal.pageSize.getHeight() - 160){
      doc.addPage();
      y = pdfContinuationBand(doc, pw, 'Relatório ' + scopeLabel + ' Consolidado') + 14;
    }
    y = pdfSectionTitle(doc, 40, y, 'Ranking de Duplas');
    const maxTotal = Math.max(...ranking.map(r=>r.total), 1);
    const barX = 300, barMaxW = pw-40-barX;
    ranking.forEach((r,i)=>{
      const ry = y + i*32;
      doc.setFillColor(...(i===0?PDF.gold:i===1?PDF.navySoft:PDF.mute));
      doc.circle(52, ry-4, 10, 'F');
      doc.setFont('helvetica','bold'); doc.setFontSize(9.5); doc.setTextColor(255,255,255);
      doc.text(String(i+1), 52, ry-1, {align:'center'});
      doc.setFont('helvetica','bold'); doc.setFontSize(10); doc.setTextColor(...PDF.text);
      doc.text(pdfFit(doc, r.name, 210), 74, ry);
      doc.setDrawColor(...PDF.border); doc.setFillColor(...PDF.grayBg);
      doc.roundedRect(barX, ry-11, barMaxW, 14, 3, 3, 'FD');
      const fillW = Math.max(6, (r.total/maxTotal) * barMaxW);
      doc.setFillColor(...PDF.green);
      doc.roundedRect(barX, ry-11, fillW, 14, 3, 3, 'F');
      doc.setFont('helvetica','bold'); doc.setFontSize(8.6); doc.setTextColor(...PDF.navy);
      doc.text(numLabel(r.total), pw-42, ry, {align:'right'});
    });
    y = y + ranking.length*32 + 10;
  }

  pdfFooter(doc);
  doc.save('Relatorio_' + scopeLabel + '_' + new Date().toISOString().slice(0,10) + '.pdf');
}

/* ======================= RENDER ROOT ======================= */
function render(){ renderSidebar(); renderMain(); }

(async function bootstrap(){
  await loadStateFromSupabase();
  render();
})();

/* Keep the greeting current if the tab is left open across a time-of-day change
   (e.g. from morning into afternoon), without re-rendering the whole page. */
let lastGreeting = greetingFor(new Date());
setInterval(()=>{
  const current = greetingFor(new Date());
  if(current !== lastGreeting){
    lastGreeting = current;
    const el = document.getElementById('heroGreeting');
    if(el) el.textContent = current + ', supervisor(a)!';
  }
}, 60000);
</script>
</body>
</html>
