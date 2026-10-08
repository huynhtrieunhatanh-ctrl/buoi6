# buoi6
using Microsoft.EntityFrameworkCore;

namespace QuanLySinhVienCRUD;

public partial class Form1 : Form
{
    private readonly AppDbContext db = new();
    private readonly TextBox txtHoTen = new();
    private readonly TextBox txtDiem = new();
    private readonly DataGridView dgvSinhVien = new();

    public Form1()
    {
        InitializeComponent();

        Text = "Quản lý Sinh viên - EF Core CRUD";
        ClientSize = new Size(760, 460);
        StartPosition = FormStartPosition.CenterScreen;

        var lblTitle = new Label
        {
            Text = "QUẢN LÝ SINH VIÊN - EF CORE (CRUD ĐẦY ĐỦ)",
            Font = new Font("Segoe UI", 14, FontStyle.Bold),
            TextAlign = ContentAlignment.MiddleCenter,
            Dock = DockStyle.Top,
            Height = 50
        };
        var lblHoTen = new Label { Text = "Họ tên:", Location = new Point(20, 68), AutoSize = true };
        var lblDiem = new Label { Text = "Điểm:", Location = new Point(330, 68), AutoSize = true };
        txtHoTen.SetBounds(80, 64, 230, 27);
        txtDiem.SetBounds(380, 64, 70, 27);

        var btnThem = TaoNut("Thêm", 20, 105, 90);
        var btnSua = TaoNut("Sửa", 120, 105, 90);
        var btnXoa = TaoNut("Xóa", 220, 105, 90);
        var btnTaiLai = TaoNut("Tải lại danh sách", 320, 105, 140);
        var btnLoc = TaoNut("Lọc SV đạt (LINQ)", 470, 105, 150);

        dgvSinhVien.SetBounds(20, 150, 720, 290);
        dgvSinhVien.Anchor = AnchorStyles.Top | AnchorStyles.Bottom | AnchorStyles.Left | AnchorStyles.Right;
        dgvSinhVien.AllowUserToAddRows = false;
        dgvSinhVien.ReadOnly = true;
        dgvSinhVien.SelectionMode = DataGridViewSelectionMode.FullRowSelect;
        dgvSinhVien.MultiSelect = false;
        dgvSinhVien.AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill;

        Controls.AddRange(new Control[]
        {
            lblTitle, lblHoTen, txtHoTen, lblDiem, txtDiem,
            btnThem, btnSua, btnXoa, btnTaiLai, btnLoc, dgvSinhVien
        });

        Load += Form1_Load;
        btnThem.Click += (s, e) => Them();
        btnSua.Click += (s, e) => Sua();
        btnXoa.Click += (s, e) => Xoa();
        btnTaiLai.Click += (s, e) => TaiLai();
        btnLoc.Click += (s, e) => Loc();
        dgvSinhVien.SelectionChanged += (s, e) => HienLenO();
    }

    private static Button TaoNut(string text, int x, int y, int w)
        => new() { Text = text, Location = new Point(x, y), Size = new Size(w, 32) };

    private void Form1_Load(object? sender, EventArgs e)
    {
        db.Database.EnsureCreated();
        if (!db.Students.Any())
        {
            db.Students.AddRange(
                new Student { FullName = "truc", Grade = 5 },
                new Student { FullName = "hoa", Grade = 9 },
                new Student { FullName = "teo", Grade = 6 });
            db.SaveChanges();
        }
        TaiLai();
    }

    private void TaiLai()
    {
        dgvSinhVien.DataSource = db.Students.AsNoTracking().OrderBy(s => s.Id).ToList();
        dgvSinhVien.ClearSelection();
    }

    private void Loc()
    {
        dgvSinhVien.DataSource = db.Students.AsNoTracking().Where(s => s.Grade >= 5).OrderBy(s => s.Id).ToList();
        dgvSinhVien.ClearSelection();
    }

    private bool DocForm(out string ten, out double diem)
    {
        ten = txtHoTen.Text.Trim();
        diem = 0;
        if (ten == "")
        {
            MessageBox.Show("Vui lòng nhập họ tên.");
            return false;
        }
        if (!double.TryParse(txtDiem.Text, out diem) || diem < 0 || diem > 10)
        {
            MessageBox.Show("Điểm phải là số từ 0 đến 10.");
            return false;
        }
        return true;
    }

    private Student? ChonSinhVien()
    {
        if (dgvSinhVien.CurrentRow?.DataBoundItem is Student sv && dgvSinhVien.SelectedRows.Count > 0)
            return sv;
        MessageBox.Show("Vui lòng chọn một sinh viên trong bảng.");
        return null;
    }

    private void Them()
    {
        if (!DocForm(out var ten, out var diem)) return;
        db.Students.Add(new Student { FullName = ten, Grade = diem });
        db.SaveChanges();
        txtHoTen.Clear();
        txtDiem.Clear();
        TaiLai();
    }

    private void Sua()
    {
        var chon = ChonSinhVien();
        if (chon == null || !DocForm(out var ten, out var diem)) return;
        var sv = db.Students.Find(chon.Id);
        if (sv == null) return;
        sv.FullName = ten;
        sv.Grade = diem;
        db.SaveChanges();
        TaiLai();
    }

    private void Xoa()
    {
        var chon = ChonSinhVien();
        if (chon == null) return;
        if (MessageBox.Show($"Xóa sinh viên {chon.FullName}?", "Xác nhận", MessageBoxButtons.YesNo) != DialogResult.Yes) return;
        var sv = db.Students.Find(chon.Id);
        if (sv == null) return;
        db.Students.Remove(sv);
        db.SaveChanges();
        txtHoTen.Clear();
        txtDiem.Clear();
        TaiLai();
    }

    private void HienLenO()
    {
        if (dgvSinhVien.SelectedRows.Count > 0 && dgvSinhVien.SelectedRows[0].DataBoundItem is Student sv)
        {
            txtHoTen.Text = sv.FullName;
            txtDiem.Text = sv.Grade.ToString();
        }
    }
}
